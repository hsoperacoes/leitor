<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1"/>
<title>Leitor de Código de Barras — HS</title>
<style>
  :root{
    --bg:#0b1220;--card:#111a2b;--txt:#e9eefc;--muted:#8aa0bf;--brand:#43b0ff;--ok:#21c55d;--warn:#f59e0b;
    --radius:16px;--shadow:0 10px 30px rgba(0,0,0,.35)
  }
  body{margin:0;background:linear-gradient(180deg,#0b1220 0%,#0d1526 70%,#0b1220 100%);color:var(--txt);font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Arial}
  .wrap{max-width:960px;margin:24px auto;padding:0 14px}
  header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:14px}
  h1{font-size:18px;margin:0}
  .panel{background:var(--card);border:1px solid rgba(255,255,255,.08);border-radius:var(--radius);box-shadow:var(--shadow);padding:14px}
  .row{display:flex;flex-wrap:wrap;gap:10px;align-items:center}
  label{font-size:13px;color:var(--muted)}
  select,button{padding:10px 12px;border-radius:12px;border:1px solid rgba(255,255,255,.12);background:#0f192b;color:var(--txt);cursor:pointer}
  button.btn{background:linear-gradient(180deg,#46b7ff 0%,#2f80ff 100%);border:none;color:#041022;font-weight:700}
  button.ghost{background:#0f192b}
  #stage{position:relative;overflow:hidden;border-radius:16px;aspect-ratio:16/9;background:#000;margin-top:12px}
  video{width:100%;height:100%;object-fit:cover}
  /* Moldura/ROI */
  .overlay{
    pointer-events:none;position:absolute;inset:0;display:grid;place-items:center
  }
  .frame{
    width:min(68%,520px);aspect-ratio:1.8/1;border:3px solid rgba(255,255,255,.9);border-radius:14px;box-shadow:0 0 0 9999px rgba(0,0,0,.50) inset;
    outline:2px dashed rgba(255,255,255,.25);outline-offset:8px;animation:glow 1.6s ease-in-out infinite
  }
  @keyframes glow{50%{border-color:rgba(255,255,255,.65)}}
  .line{position:absolute;left:50%;top:50%;width:60%;height:2px;background:rgba(67,176,255,.9);transform:translate(-50%,-50%);box-shadow:0 0 10px rgba(67,176,255,.8)}
  .bar{display:inline-block;padding:.3rem .6rem;border-radius:8px;background:#0f1a31;border:1px solid rgba(255,255,255,.12);font-size:13px}
  .ok{color:#0b1b12;border-color:rgba(33,197,93,.45);background:rgba(33,197,93,.2)}
  .warn{color:#201402;border-color:rgba(245,158,11,.45);background:rgba(245,158,11,.18)}
  #result{margin-top:10px;font-family:ui-monospace,monospace;font-size:14px}
  .muted{color:var(--muted);font-size:12px}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>Leitor de Código de Barras — HS</h1>
    <div class="muted">Sem licença, com fallback automático</div>
  </header>

  <div class="panel">
    <div class="row" style="gap:8px 12px">
      <div>
        <label for="cameraSel">Câmera</label><br/>
        <select id="cameraSel"></select>
      </div>
      <div>
        <label for="formats">Formatos</label><br/>
        <select id="formats" multiple size="4" style="min-width:210px">
          <option value="qr_code">QR Code</option>
          <option value="ean_13" selected>EAN-13</option>
          <option value="ean_8">EAN-8</option>
          <option value="code_128" selected>Code 128</option>
          <option value="code_39">Code 39</option>
          <option value="upc_a">UPC-A</option>
          <option value="upc_e">UPC-E</option>
          <option value="itf">ITF</option>
          <option value="pdf417">PDF417</option>
          <option value="data_matrix">Data Matrix</option>
          <option value="aztec">Aztec</option>
          <option value="codabar">Codabar</option>
        </select>
      </div>
      <div>
        <label>ROI</label><br/>
        <span class="bar">Ativo (janela central)</span>
      </div>
      <div>
        <label>Lanterna</label><br/>
        <button id="torchBtn" class="ghost" disabled>Ativar</button>
      </div>
      <div style="margin-left:auto;display:flex;gap:8px">
        <button id="startBtn" class="btn">Iniciar</button>
        <button id="stopBtn" class="ghost" disabled>Parar</button>
      </div>
    </div>

    <div id="stage">
      <video id="video" playsinline></video>
      <div class="overlay">
        <div class="frame"></div>
        <div class="line"></div>
      </div>
    </div>

    <div id="result">
      <div><span id="status" class="bar warn">Parado</span></div>
      <div style="margin-top:6px">Último código: <span id="codeTxt" class="bar">—</span></div>
      <div class="muted" id="tip" style="margin-top:6px">Dica: aproxime o código da moldura; mantenha o enquadramento estável.</div>
    </div>
  </div>
</div>

<!-- ZXing fallback (WASM/JS) -->
<script type="module">
  // Carrega ZXing dinamicamente só se precisar
  let ZXING = null;

  const els = {
    video: document.getElementById('video'),
    cameraSel: document.getElementById('cameraSel'),
    formats: document.getElementById('formats'),
    startBtn: document.getElementById('startBtn'),
    stopBtn: document.getElementById('stopBtn'),
    torchBtn: document.getElementById('torchBtn'),
    status: document.getElementById('status'),
    codeTxt: document.getElementById('codeTxt'),
    tip: document.getElementById('tip')
  };

  let stream = null;
  let track = null;
  let imageCapture = null;
  let useBarcodeDetector = ('BarcodeDetector' in window);
  let detector = null;
  let rafId = null;
  let running = false;

  // Anti-spam: confirma o MESMO código duas vezes seguidas antes de disparar o callback
  let lastValue = null, stableCount = 0;

  // Canvas para ROI
  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d', { willReadFrequently: true });

  // Prepara lista de câmeras
  async function listCameras(){
    const devices = await navigator.mediaDevices.enumerateDevices();
    const cams = devices.filter(d => d.kind === 'videoinput');
    els.cameraSel.innerHTML = '';
    cams.forEach((c, i) => {
      const opt = document.createElement('option');
      opt.value = c.deviceId;
      opt.textContent = c.label || `Câmera ${i+1}`;
      els.cameraSel.appendChild(opt);
    });
  }

  // Obtém formatos selecionados
  function selectedFormats(){
    return Array.from(els.formats.selectedOptions).map(o => o.value);
  }

  // Tocar beep
  function beep(){
    try{
      const a = new (window.AudioContext || window.webkitAudioContext)();
      const o = a.createOscillator(); const g = a.createGain();
      o.type='sine'; o.frequency.value=880; o.connect(g); g.connect(a.destination);
      o.start(); g.gain.exponentialRampToValueAtTime(0.0001, a.currentTime+0.18);
      setTimeout(()=>{o.stop(); a.close();}, 200);
    }catch{}
  }

  // Vibração
  function vib(){
    if(navigator.vibrate) navigator.vibrate(80);
  }

  // Callback quando confirmar código
  function onCode(value, format){
    els.codeTxt.textContent = `${value} (${format})`;
    els.status.textContent = 'Capturado';
    els.status.className = 'bar ok';
    beep(); vib();
    stop(); // auto-fecha como no Dynamsoft aprovado por você
    // Aqui você pode disparar seu fluxo:
    // window.postMessage({ type:'HS_CODE', value, format }, '*');
  }

  // Liga/desliga lanterna
  async function toggleTorch(){
    if(!track) return;
    const caps = track.getCapabilities?.() || {};
    if(!('torch' in caps)) return;
    const settings = track.getSettings?.() || {};
    const want = !settings.torch;
    await track.applyConstraints({ advanced: [{ torch: want }]});
    els.torchBtn.textContent = want ? 'Desativar' : 'Ativar';
  }

  els.torchBtn.addEventListener('click', toggleTorch);

  // Loop de leitura
  async function loop(){
    if(!running) return;

    // Define ROI central (proporção similar ao retângulo de mira)
    const vw = els.video.videoWidth, vh = els.video.videoHeight;
    if(!vw || !vh){ rafId = requestAnimationFrame(loop); return; }

    const roiW = Math.floor(vw * 0.68);
    const roiH = Math.floor(roiW / 1.8); // mesmo aspect da moldura
    const sx = Math.floor((vw - roiW)/2);
    const sy = Math.floor((vh - roiH)/2);

    // Desenha ROI no canvas
    canvas.width = roiW; canvas.height = roiH;
    ctx.drawImage(els.video, sx, sy, roiW, roiH, 0, 0, roiW, roiH);

    let value = null, format = 'desconhecido';

    try{
      if(useBarcodeDetector && detector){
        const bitmap = await createImageBitmap(canvas);
        const codes = await detector.detect(bitmap);
        bitmap.close?.();

        if(codes && codes.length){
          // pega o melhor
          const c = codes[0];
          value = c.rawValue || c.data || null;
          format = c.format || 'detector';
        }
      }else if(ZXING){
        // ZXing
        if(!ZXING.reader){
          ZXING.reader = new ZXING.BrowserMultiFormatReader();
          // Dica: limitar formatos acelera. Vamos mapear a seleção para ZXing.
          const hints = new ZXING.Hints();
          const Format = ZXING.BarcodeFormat;
          const selected = mapFormatsToZXing(selectedFormats(), Format);
          hints.set(ZXING.DecodeHintType.POSSIBLE_FORMATS, selected);
          ZXING.reader.setHints(hints);
        }
        const luminance = ZXING.HTMLCanvasElementLuminanceSourceFactory.createFromCanvas(canvas);
        const binaryBitmap = new ZXING.BinaryBitmap(new ZXING.HybridBinarizer(luminance));
        const res = ZXING.reader.decode(binaryBitmap);
        if(res){
          value = res.getText();
          format = res.getBarcodeFormat().toString();
        }
      }
    }catch(_){ /* normal falhar, só tenta de novo */ }

    if(value){
      if(value === lastValue) stableCount++; else { lastValue = value; stableCount = 1; }
      els.status.textContent = `Lendo… (${stableCount}/2)`;
      els.status.className = 'bar';
      if(stableCount >= 2){
        onCode(value, format);
        return;
      }
    }else{
      els.status.textContent = 'Procurando código…';
      els.status.className = 'bar warn';
      lastValue = null; stableCount = 0;
    }

    rafId = requestAnimationFrame(loop);
  }

  function mapFormatsToZXing(list, Format){
    // Mapeamento básico
    const map = {
      'qr_code': Format.QR_CODE,
      'ean_13': Format.EAN_13,
      'ean_8': Format.EAN_8,
      'code_128': Format.CODE_128,
      'code_39': Format.CODE_39,
      'upc_a': Format.UPC_A,
      'upc_e': Format.UPC_E,
      'itf': Format.ITF,
      'pdf417': Format.PDF_417,
      'data_matrix': Format.DATA_MATRIX,
      'aztec': Format.AZTEC,
      'codabar': Format.CODABAR
    };
    const out = [];
    for(const f of list){ if(map[f]) out.push(map[f]); }
    return out.length ? out : [Format.EAN_13, Format.CODE_128, Format.QR_CODE];
  }

  async function start(){
    if(running) return;

    // Permissão + stream
    const constraints = {
      audio: false,
      video: {
        deviceId: els.cameraSel.value ? { exact: els.cameraSel.value } : undefined,
        facingMode: 'environment',
        width: { ideal: 1280 },
        height:{ ideal: 720 }
      }
    };

    stream = await navigator.mediaDevices.getUserMedia(constraints);
    els.video.srcObject = stream;
    await els.video.play();

    track = stream.getVideoTracks()[0];
    els.stopBtn.disabled = false;
    els.startBtn.disabled = true;

    // Lanterna
    try{
      imageCapture = new ImageCapture(track);
      const caps = track.getCapabilities?.() || {};
      els.torchBtn.disabled = !('torch' in caps);
    }catch{ els.torchBtn.disabled = true; }

    // Detector ou ZXing
    if(useBarcodeDetector){
      try{
        detector = new BarcodeDetector({ formats: selectedFormats() });
      }catch{
        useBarcodeDetector = false;
      }
    }
    if(!useBarcodeDetector && !ZXING){
      // Carrega ZXing on-demand
      ZXING = await import('https://unpkg.com/@zxing/library@0.20.0/esm/index.js');
    }

    running = true;
    els.status.textContent = 'Procurando código…';
    els.status.className = 'bar warn';
    loop();
  }

  async function stop(){
    running = false;
    if(rafId) cancelAnimationFrame(rafId);
    if(stream){
      stream.getTracks().forEach(t => t.stop());
      stream = null; track = null; imageCapture = null;
    }
    els.video.srcObject = null;
    els.startBtn.disabled = false;
    els.stopBtn.disabled = true;
    els.torchBtn.disabled = true;
    els.status.textContent = 'Parado';
    els.status.className = 'bar warn';
  }

  // UI
  els.startBtn.addEventListener('click', start);
  els.stopBtn.addEventListener('click', stop);

  // Recria detector ao mudar formatos
  els.formats.addEventListener('change', () => {
    if(useBarcodeDetector){
      try{ detector = new BarcodeDetector({ formats: selectedFormats() }); }catch{}
    }else{
      // Reseta hints do ZXing numa próxima leitura
      if(ZXING){ ZXING.reader = null; }
    }
  });

  // Atualiza câmeras ao entrar
  (async () => {
    try{
      // Necessário pedir permissão pelo menos 1x para liberar labels
      await navigator.mediaDevices.getUserMedia({ video: true, audio: false }).then(s => s.getTracks().forEach(t=>t.stop()));
    }catch{}
    await listCameras();
  })();
</script>
</body>
</html>
