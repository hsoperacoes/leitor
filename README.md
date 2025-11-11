<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>GERENCIAL HS - Sistema Unificado</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" />
  <script src="https://unpkg.com/@ericblade/quagga2@1.2.7/dist/quagga.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-javascript-barcode@9.6.21/dist/dbr.js"></script>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: Arial, sans-serif; background-color: #000; color: #fff;
      margin: 0; padding: 0; display: flex; flex-direction: column; align-items: center; min-height: 100vh;
    }
    .logo-topo { width: 100px; height: auto; margin: 30px 0 10px; }
    .container {
      width: 100%; max-width: 900px; background-color: #2c2c2c; padding: 40px 30px;
      border-radius: 12px; box-shadow: 0 0 10px rgba(0,0,0,0.5); margin-bottom: 20px;
    }
    .form-link {
      display:flex; align-items:center; justify-content:center; background-color:#4CAF50; color:#fff;
      padding:15px; margin:10px 0; border-radius:6px; font-size:16px; text-decoration:none;
      transition:background-color .3s ease; cursor:pointer; border:none; width:100%;
    }
    .form-link:hover{ background-color:#45a049; } .form-link i{ margin-right:10px; }

    .back-button{
      position:fixed; top:20px; left:20px; background-color:#4CAF50; color:#fff; border:none; padding:10px 15px;
      border-radius:6px; cursor:pointer; font-size:14px; z-index:1000; display:none; transition:background-color .3s ease;
    }
    .back-button:hover{ background-color:#45a049; } .back-button.show{ display:block; }

    .form-container{
      max-width:600px; background:#1e1e1e; padding:30px; border-radius:8px; box-shadow:0 2px 10px rgba(0,0,0,.3); color:#fff; margin:0 auto;
    }
    .form-container h1,.form-container h2{ text-align:center; color:#fff; margin-bottom:30px; }
    .form-group{ margin-bottom:20px; }
    label{ font-size:14px; font-weight:600; display:block; margin-bottom:5px; color:#ccc; }
    .question-title{ font-size:16px; font-weight:bold; color:#fff; margin-bottom:5px; display:block; }

    input,select,textarea{
      width:100%; padding:12px; font-size:14px; border:1px solid #444; border-radius:4px; background-color:#2a2a2a; color:#fff; box-sizing:border-box;
    }
    input:focus,select:focus,textarea:focus{ outline:none; border-color:#673ab7; box-shadow:0 0 0 2px rgba(103,58,183,.2); }
    button{ background-color:#673ab7; color:#fff; border:none; padding:12px 20px; border-radius:4px; font-size:16px; cursor:pointer; font-weight:600; transition:background-color .3s; }
    button:hover{ background-color:#5e35b1; }

    .submit-buttons{ display:flex; justify-content:space-between; margin-top:20px; gap:10px; }
    .submit-button,.clear-button{ padding:12px 20px; font-size:14px; font-weight:600; border:none; border-radius:4px; cursor:pointer; flex:1; }
    .submit-button{ background-color:#673ab7; color:#fff; }
    .clear-button{ background-color:#5f6368; color:#fff; }
    .submit-button:hover{ background-color:#5e35b1; } .clear-button:hover{ background-color:#3c4043; }

    .gerador-section .form-container{ background-color:#1e1e1e; color:#fff; max-width:800px; }
    .gerador-section .input-area,.gerador-section .output-area{ background:#1e1e1e; color:#fff; }
    .controls{ display:flex; gap:10px; margin:20px 0; flex-wrap:wrap; }
    .controls button{ flex:1; min-width:120px; }
    .gerador-section .controls button:nth-child(1){ background-color:#4CAF50; }
    .gerador-section .controls button:nth-child(1):hover{ background-color:#45a049; }
    .gerador-section .controls button:nth-child(2){ background-color:#f44336; }
    .gerador-section .controls button:nth-child(2):hover{ background-color:#d32f2f; }
    .gerador-section .controls button:nth-child(3){ background-color:#ff9800; }
    .gerador-section .controls button:nth-child(3):hover{ background-color:#e68a00; }
    .gerador-section .controls button:nth-child(4){ background-color:#2196F3; }
    .gerador-section .controls button:nth-child(4):hover{ background-color:#0b7dda; }
    .gerador-section h1,.gerador-section h2{ color:#fff; }
    .gerador-section .example-title{ color:#ccc; } .gerador-section .barcode-count{ color:#fff; }

    .barcode-container{ display:grid; grid-template-columns:repeat(auto-fit,minmax(200px,1fr)); gap:15px; margin-top:20px; }
    .barcode-item{ background:#2a2a2a; border:1px solid #444; padding:10px; border-radius:4px; text-align:center; }
    svg text{ fill:#fff !important; }

    #success-message,#error-message{ display:none; padding:12px; margin-top:20px; border-radius:4px; text-align:center; font-weight:500; }
    #success-message{ background-color:#e8f5e8; color:#137333; border:1px solid #ceead6; }
    #error-message{ background-color:#fce8e6; color:#d93025; border:1px solid #f9dedc; }
    .loading{ text-align:center; font-style:italic; margin-top:10px; color:#1a73e8; }

    .loading-overlay{
      display:none; position:fixed; top:0; left:0; right:0; bottom:0; background-color:rgba(0,0,0,.5);
      color:#fff; justify-content:center; align-items:center; flex-direction:column; z-index:1000;
    }
    .spinner{ border:4px solid transparent; border-top:4px solid #fff; border-radius:50%; width:40px; height:40px; animation:spin 2s linear infinite; }
    @keyframes spin{ 0%{transform:rotate(0)} 100%{transform:rotate(360deg)} }

    .barcode-count{ font-size:16px; margin-top:10px; font-weight:bold; color:#fff; }
    .count-number{ color:#ea4335; font-weight:bold; }
    .required-star{ color:#ea4335; }
    .hidden{ display:none; }
    .logout{ text-align:right; margin-bottom:10px; }
    .logout button{ padding:8px 16px; background-color:#5f6368; font-size:14px; }
    .history button{ background-color:#ea4335; margin-top:15px; }
    .logo{ display:block; margin:20px auto 10px; max-width:80px; }

    #reader{ width:100%; max-width:350px; height:240px; margin:10px auto; position:relative; }
    #interactive.viewport{ position:relative; width:100%; height:100%; }
    canvas,video{ max-width:100%; width:100% !important; height:auto !important; }

    footer{
      text-align:center; font-size:14px; color:#ccc; background-color:#2c2c2c; padding:15px; border-radius:6px;
      margin:20px auto; max-width:900px; width:100%;
    }

    .section{ display:none; width:100%; padding:20px; justify-content:center; align-items:center; flex-direction:column; }
    .section.active{ display:flex; }
    .checkbox-label{ font-size:14px; display:flex; align-items:center; gap:8px; margin-bottom:20px; }
    input[readonly]{ background-color:#2a2a2a; color:#bbb; }
    #grupo-tamanho-g{ display:none; }

    /* === Scanner tela cheia com mira === */
    #scanner{
      width:100%; height:100vh; position:fixed; top:0; left:0; z-index:1000; display:none;
      justify-content:center; align-items:center; flex-direction:column; background-color:#000;
    }
    .scanner-header{
      width:100%; max-width:800px; padding:8px 12px; display:flex; justify-content:space-between; align-items:center;
    }
    .scanner-actions button{ margin-left:8px; background:#333; }
    .aimbox{
      position:absolute; box-shadow: 0 0 0 200vmax rgba(0,0,0,.45) inset;
      width:65vmin; max-width:80%; aspect-ratio: 1 / 1; border:2px solid #43b0ff; border-radius:16px;
      pointer-events:none;
    }
    .hint{ margin-top:10px; color:#9ecbff; font-size:14px; }

    @media print{
      .back-button,.input-area,.no-print,button,.controls,.example-title{ display:none !important; }
      .output-area{ width:100%; background:none; box-shadow:none; padding:0; margin:0; }
      .barcode-container{ display:grid; grid-template-columns:repeat(3,1fr); gap:10px; margin:0; }
      .barcode-item{ margin:0; padding:5px; box-shadow:none; background:none; break-inside:avoid; page-break-inside:avoid; }
      svg{ background:none; }
      svg text{ fill:#000 !important; font-size:14px !important; }
    }

    @media (max-width:768px){
      .form-container{ margin:20px 10px; padding:20px; }
      .submit-buttons{ flex-direction:column; gap:10px; }
      .controls{ flex-direction:column; }
      .controls button{ margin:5px 0; }
      .container{ padding:20px 15px; }
    }
  </style>
</head>
<body>
  <button class="back-button" id="backButton" onclick="voltarHome()">
    <i class="fas fa-arrow-left"></i> Voltar à Home
  </button>

  <div id="home" class="section active">
    <img src="logo.png" alt="Logo HS" class="logo-topo" />
    <div class="container">
      <button class="form-link" onclick="mostrarSecao('folgas')"><i class="fas fa-calendar-plus"></i> CADASTRO DE FOLGAS</button>
      <button class="form-link" onclick="mostrarSecao('falta')"><i class="fas fa-calendar-minus"></i> CADASTRO DE FALTA</button>
      <button class="form-link" onclick="mostrarSecao('sacola')"><i class="fas fa-lock"></i> CONTAGEM DE SACOLA</button>
      <button class="form-link" onclick="mostrarSecao('divergencia')"><i class="fas fa-file-invoice-dollar"></i> DIVERGÊNCIA DE NOTAS FISCAIS</button>
      <button class="form-link" onclick="mostrarSecao('transferencia')"><i class="fas fa-exchange-alt"></i> TRANSFERÊNCIA ENTRE LOJAS</button>
      <button class="form-link" onclick="mostrarSecao('gerador')"><i class="fas fa-barcode"></i> GERADOR DE CÓDIGO DE BARRAS</button>
      <button class="form-link" onclick="mostrarSecao('nf')"><i class="fas fa-receipt"></i> RECEBIMENTO DE NOTA FISCAL</button>
    </div>
  </div>

  <!-- FOLGAS -->
  <div id="folgas" class="section">
    <div class="form-container">
      <img src="logo.png" alt="Logo" class="logo">
      <h2>Cadastro de Folga Funcionários</h2>
      <form id="form-folgas" method="POST" action="https://script.google.com/macros/s/AKfycbwh-YUwL2o3_i-bfcV9RMzLcoI98vyyGwEXf4LHlG5KJ59gIAlUe1_VVlFQMBqU6PwR/exec">
        <div class="form-group">
          <label for="filial-folgas">Filial</label>
          <select id="filial-folgas" name="filial" required onchange="atualizarFuncionarios()">
            <option value="">Selecione uma filial</option>
            <option value="ARTUR">ARTUR</option><option value="FLORIANO">FLORIANO</option>
            <option value="JOTA">JOTA</option><option value="MODA">MODA</option><option value="PONTO">PONTO</option>
          </select>
        </div>
        <div class="form-group">
          <label for="funcionario-folgas">Funcionário</label>
          <select id="funcionario-folgas" name="funcionario" required>
            <option value="">Selecione a filial primeiro</option>
          </select>
        </div>
        <div class="form-group">
          <label for="dataTrabalho-folgas">Dia Trabalhado</label>
          <input type="date" id="dataTrabalho-folgas" name="dataTrabalho" required>
        </div>
        <div class="form-group">
          <label for="motivo-folgas">Motivo da Folga</label>
          <select id="motivo-folgas" name="motivo" required>
            <option value="">Selecione um motivo</option>
            <option value="DOMINGO">DOMINGO</option><option value="FERIADO">FERIADO</option><option value="OUTROS">OUTROS</option>
          </select>
        </div>
        <div class="form-group" id="motivoOutros-folgas" style="display: none;">
          <label for="outrosMotivo-folgas">Especificar o Motivo</label>
          <input type="text" id="outrosMotivo-folgas" name="outrosMotivo" placeholder="Escreva o motivo">
        </div>
        <div class="form-group">
          <label for="dataFolga-folgas">Data da Folga</label>
          <input type="date" id="dataFolga-folgas" name="dataFolga" required>
        </div>
        <button type="submit">Enviar</button>
      </form>
    </div>
  </div>

  <!-- FALTA -->
  <div id="falta" class="section">
    <div class="form-container">
      <img src="logo.png" alt="Logo" class="logo">
      <h2>Cadastro de Falta</h2>
      <form id="form-falta">
        <div class="form-group">
          <label for="filial-falta">Filial <span class="required-star">*</span></label>
          <select id="filial-falta" name="filial" required>
            <option value="">Selecione uma filial</option>
            <option value="ARTUR">ARTUR</option><option value="FLORIANO">FLORIANO</option>
            <option value="JOTA">JOTA</option><option value="MODA">MODA</option><option value="PONTO">PONTO</option>
          </select>
        </div>
        <div class="form-group">
          <label for="funcionario-falta">Funcionário <span class="required-star">*</span></label>
          <select id="funcionario-falta" name="funcionario" required>
            <option value="">Selecione um funcionário</option>
          </select>
        </div>
        <div class="form-group">
          <label for="data-falta">Data da Falta <span class="required-star">*</span></label>
          <input type="date" id="data-falta" name="data_falta" required>
        </div>
        <div class="form-group">
          <label for="motivo-falta">Motivo <span class="required-star">*</span></label>
          <select id="motivo-falta" name="motivo" required>
            <option value="">Selecione um motivo</option>
            <option value="ATESTADO MÉDICO">ATESTADO MÉDICO</option>
            <option value="FALTA INJUSTIFICADA">FALTA INJUSTIFICADA</option>
          </select>
        </div>
        <div class="form-group" id="dias-afastamento-container" style="display: none;">
          <label for="observacao-falta">Dias de afastamento (caso atestado)</label>
          <textarea id="observacao-falta" name="observacao" rows="4" placeholder="Ex: 3 DIAS 12/04 A 14/04"></textarea>
        </div>
        <button type="submit">Enviar</button>
      </form>
    </div>
  </div>

  <!-- SACOLA -->
  <div id="sacola" class="section">
    <div class="form-container">
      <img src="logo.png" alt="Logo" class="logo">
      <h2>Contagem de Sacola</h2>
      <form id="form-sacola">
        <div class="form-group">
          <label for="filial">Filial <span class="required-star">*</span></label>
          <select id="filial" name="filial" required>
            <option value="">Selecione uma filial</option>
            <option value="ARTUR">ARTUR</option><option value="FLORIANO">FLORIANO</option>
            <option value="JOTA">JOTA</option><option value="MODA">MODA</option><option value="PONTO">PONTO</option>
          </select>
        </div>
        <div class="form-group">
          <label for="data-contagem">Data da Contagem <span class="required-star">*</span></label>
          <input type="text" id="data-contagem" name="data_contagem" readonly required>
        </div>
        <div class="form-group"><label for="sacola-p">Sacola P <span class="required-star">*</span></label><input type="number" id="sacola-p" name="sacola_p" placeholder="Quantidade" min="0" required></div>
        <div class="form-group"><label for="envelope-p">Envelope P <span class="required-star">*</span></label><input type="number" id="envelope-p" name="envelope_p" placeholder="Quantidade" min="0" required></div>
        <div class="form-group"><label for="sacola-m">Sacola M <span class="required-star">*</span></label><input type="number" id="sacola-m" name="sacola_m" placeholder="Quantidade" min="0" required></div>
        <div class="form-group"><label for="envelope-m">Envelope M <span class="required-star">*</span></label><input type="number" id="envelope-m" name="envelope_m" placeholder="Quantidade" min="0" required></div>
        <div class="form-group checkbox-label">
          <input type="checkbox" id="mostrar-g" onclick="toggleCamposG()"><label for="mostrar-g">Possui Sacola e Envelope G</label>
        </div>
        <div id="grupo-tamanho-g">
          <div class="form-group"><label for="sacola-g">Sacola G <span class="required-star">*</span></label><input type="number" id="sacola-g" name="sacola_g" placeholder="Quantidade" min="0"></div>
          <div class="form-group"><label for="envelope-g">Envelope G <span class="required-star">*</span></label><input type="number" id="envelope-g" name="envelope_g" placeholder="Quantidade" min="0"></div>
        </div>
        <button type="submit" id="btn-enviar">Enviar</button>
      </form>
    </div>
  </div>

  <!-- DIVERGÊNCIA -->
  <div id="divergencia" class="section">
    <div class="form-container">
      <img src="logo.png" alt="Logo" class="logo">
      <h2>Divergências em Notas Fiscais</h2>
      <form id="formulario-divergencia" onsubmit="enviarFormularioDivergencia(event)">
        <div class="form-group">
          <label>Filial</label>
          <select name="filial" required>
            <option value="">Selecione uma filial</option>
            <option value="ARTUR">ARTUR</option><option value="FLORIANO">FLORIANO</option>
            <option value="JOTA">JOTA</option><option value="MODA">MODA</option><option value="PONTO">PONTO</option>
          </select>
        </div>
        <div class="form-group">
          <label>Transportadora</label>
          <select name="transportadora" id="transportadora-div" required>
            <option value="BRASPRESS">BRASPRESS</option><option value="OUTROS">OUTROS</option>
          </select>
        </div>
        <div class="form-group" id="outrosTransportadoraDiv" style="display: none;">
          <label for="outraTransportadoraDiv">Qual é a Transportadora?</label>
          <input type="text" id="outraTransportadoraDiv" name="outraTransportadora">
        </div>
        <div class="form-group"><label for="dataRecebimentoDiv">Data de Recebimento</label><input type="date" id="dataRecebimentoDiv" name="dataRecebimento" required></div>
        <div class="form-group"><label for="notaFiscalDiv">Número da Nota Fiscal</label><input type="text" id="notaFiscalDiv" name="notaFiscal" required></div>
        <div class="form-group"><label for="serieNotaDiv">Série da Nota Fiscal</label><input type="text" id="serieNotaDiv" name="serieNota" required></div>
        <div class="form-group"><label for="referenciaDiv">Referência</label><input type="text" id="referenciaDiv" name="referencia" maxlength="4" required></div>
        <div class="form-group"><label for="corDiv">Cor</label><input type="text" id="corDiv" name="cor" maxlength="6" required></div>
        <div class="form-group"><label for="tamanhoDiv">Tamanho</label><input type="text" id="tamanhoDiv" name="tamanho" maxlength="3" required></div>
        <div class="form-group"><label for="quantidadeDiv">Quantidade</label><input type="number" id="quantidadeDiv" name="quantidade" required></div>
        <div class="form-group">
          <label>Divergência</label>
          <select name="divergencia" required>
            <option value="">Selecione uma opção</option>
            <option value="MERCADORIA PASSANDO">MERCADORIA PASSANDO</option>
            <option value="MERCADORIA FALTANDO">MERCADORIA FALTANDO</option>
          </select>
        </div>
        <div class="form-group"><button type="submit">Enviar</button></div>
      </form>
      <div id="loadingMessageDiv" class="loading" style="display: none;">Enviando... Por favor, aguarde.</div>
    </div>
  </div>

  <!-- TRANSFERÊNCIA -->
  <div id="transferencia" class="section">
    <div class="form-container">
      <img src="logo.png" alt="Logo" class="logo">
      <h1>TRANSFERÊNCIA ENTRE LOJAS</h1>
      <form id="transfer-form">
        <div class="form-group">
          <div class="question-title">Email da filial de origem:(Preenchimento automatico) <span class="required-star">*</span></div>
          <input type="email" name="email" id="email-trans" required readonly>
        </div>
        <div class="form-group">
          <div class="question-title">FILIAL ORIGEM <span class="required-star">*</span></div>
          <select name="filialOrigem" id="filial-origem" required onchange="atualizarEmailTrans()">
            <option value="" disabled selected>Selecione</option>
            <option value="ARTUR">ARTUR</option><option value="FLORIANO">FLORIANO</option>
            <option value="JOTA">JOTA</option><option value="MODA">MODA</option>
            <option value="PONTO">PONTO</option><option value="JA">JA</option><option value="JE">JE</option>
          </select>
        </div>
        <div class="form-group">
          <div class="question-title">FILIAL DESTINO <span class="required-star">*</span></div>
          <select name="filialDestino" id="filial-destino" required>
            <option value="" disabled selected>Selecione</option>
            <option value="ARTUR">ARTUR</option><option value="FLORIANO">FLORIANO</option>
            <option value="JOTA">JOTA</option><option value="MODA">MODA</option>
            <option value="PONTO">PONTO</option><option value="JA">JA</option><option value="JE">JE</option>
          </select>
        </div>
        <div class="form-group">
          <div class="question-title">MERCADORIAS QUE ESTÃO SAINDO <span class="required-star">*</span></div>
          <textarea name="mercadorias" id="mercadorias-trans" required placeholder="Inclua aqui os códigos de barras." oninput="contarLinhasTrans()"></textarea>
        </div>
        <div id="barcode-count-trans" class="barcode-count">Total de itens: <span id="total-itens-trans" class="count-number">0</span></div>
        <div id="success-message-trans" style="display:none;background-color:#e8f5e8;color:#137333;padding:12px;margin-top:20px;border-radius:4px;text-align:center;border:1px solid #ceead6;">Transferência enviada com sucesso!</div>
        <div id="error-message-trans" style="display:none;background-color:#fce8e6;color:#d93025;padding:12px;margin-top:20px;border-radius:4px;text-align:center;border:1px solid #f9dedc;"></div>
        <div class="submit-buttons">
          <button type="reset" class="clear-button">Limpar formulário</button>
          <button type="submit" class="submit-button" id="submit-button-trans">Enviar</button>
        </div>
      </form>
      <div id="numero-transferencia" style="display:none;margin-top:20px;font-size:18px;text-align:center;">Número da transferência: <strong id="transfer-id"></strong></div>
    </div>
    <div class="loading-overlay" id="loading-overlay-trans">
      <div class="loading-content">
        <div class="spinner"></div>
        <h3>Processando sua transferência...</h3>
        <p>Aguarde enquanto enviamos os dados.</p>
      </div>
    </div>
  </div>

  <!-- GERADOR EAN13 -->
  <div id="gerador" class="section gerador-section">
    <div class="form-container">
      <img src="logo.png" alt="Logo" class="logo">
      <h1>Gerador de Códigos de Barras em Lote - EAN13</h1>
      <div class="input-area">
        <h2>Cole seus códigos (um por linha):</h2>
        <textarea id="codigos-barras" placeholder="Cole aqui vários códigos EAN13, um por linha&#10;Exemplo:&#10;7891000315507&#10;7891910000197&#10;7891234567890"></textarea>
        <div class="controls">
          <button onclick="gerarTodosBarras()">Gerar Códigos</button>
          <button onclick="limparTudoBarras()">Limpar Tudo</button>
          <button onclick="copiarCodigosBarras()">Copiar Códigos</button>
          <button onclick="window.print()">Imprimir Códigos</button>
        </div>
        <div class="example-title">Exemplos (não são apagados ao limpar):</div>
      </div>
      <div class="output-area">
        <h2>Códigos Gerados:</h2>
        <div id="barcodes" class="barcode-container"></div>
      </div>
    </div>
  </div>

  <!-- NF-e -->
  <div id="nf" class="section">
    <div class="form-container">
      <img src="logo.png" alt="Logo" class="logo">
      <div id="login-nf">
        <h2>Login da Filial</h2>
        <label for="codigo-nf">Código da Filial</label>
        <input type="text" id="codigo-nf" placeholder="Digite sua senha" />
        <button onclick="entrarNF()">Entrar</button>
      </div>

      <div id="principal-nf" class="hidden">
        <div class="logout"><button onclick="sairNF()">Sair</button></div>
        <h2>Consulta de Nota Fiscal</h2>
        <label for="chave-nf">Chave de Acesso (44 dígitos)</label>
        <input type="text" id="chave-nf" placeholder="Digite a chave completa" maxlength="44" />
        <div class="scanner-actions" style="margin:8px 0;">
          <button onclick="abrirLeitorNF()">📷 Ler Código</button>
        </div>

        <!-- Tela do Scanner -->
        <div id="scanner">
          <div class="scanner-header">
            <div><strong>Leitor de Código</strong></div>
            <div class="scanner-actions">
              <button id="torch-btn" onclick="toggleTorchNF()" class="hidden">🔦 Lanternar</button>
              <button onclick="fecharLeitorNF()">✖ Fechar</button>
            </div>
          </div>
          <div id="scanner-video" style="position:relative; width:100%; max-width:800px; aspect-ratio:16/9;"></div>
          <div class="aimbox"></div>
          <div class="hint">Centralize o QR/código dentro da moldura azul</div>
        </div>

        <button onclick="consultarNotaNF()">Consultar</button>
        <div id="loading-nf" class="loading hidden">⏳ Consultando nota fiscal...</div>
        <div id="resultado-nf" class="result hidden"></div>
        <div id="erro-nf" style="color:#d93025;margin-top:10px;font-size:14px;"></div>

        <div class="history">
          <h3>Histórico da Filial</h3>
          <ul id="historicoLista-nf"></ul>
          <button onclick="limparHistoricoLocalNF()">🗑 Limpar Histórico Local</button>
        </div>
      </div>
    </div>
  </div>

  <footer>HS Operações © 2025 - Todos os direitos reservados</footer>

  <script>
    // ====== LICENÇA DYNAMSOFT ======
    Dynamsoft.DBR.BarcodeScanner.license = "DLS2eyJoYW5kc2hha2VDb2RlIjoiMTA0NDIzNjk5LU1UQTBOREl6TmprNUxYZGxZaTFVY21saGJGQnliMm8iLCJtYWluU2VydmVyVVJMIjoiaHR0cHM6Ly9tZGxzLmR5bmFtc29mdG9ubGluZS5jb20iLCJvcmdhbml6YXRpb25JRCI6IjEwNDQyMzY5OSIsInN0YW5kYnlTZXJ2ZXJVUkwiOiJodHRwczovL3NkbHMuZHluYW1zb2Z0b25saW5lLmNvbSIsImNoZWNrQ29kZSI6MTEzNzY0MDQxN30";

    // ====== NAVEGAÇÃO ======
    function mostrarSecao(id){
      document.querySelectorAll(".section").forEach(s=>s.classList.remove('active'));
      document.getElementById(id).classList.add('active');
      document.getElementById('backButton').classList.add('show');
      window.scrollTo({top:0,behavior:'smooth'});
    }
    function voltarHome(){
      document.querySelectorAll(".section").forEach(s=>s.classList.remove('active'));
      document.getElementById('home').classList.add('active');
      document.getElementById('backButton').classList.remove('show');
      window.scrollTo({top:0,behavior:'smooth'});
    }

    // ====== DADOS FUNCIONÁRIOS ======
    const funcionariosPorFilial = {
      "ARTUR": ["LUCILENE","POLIANA","TAINARA"],
      "FLORIANO": ["IOLANDA","MEIRE","SARA","GABRYELLA"],
      "JOTA": ["BRUNO","CARINA","DENISE","FABIOLA","INGRID","MARCOS","RAYSSA","VERA"],
      "MODA": ["ANA CLARA","DAIANE","JOANA","MARCIA","MARIA"],
      "PONTO": ["DANIELA","EVANEUZA","ISADORA","PAULA","PRISCILA","SÔNIA","SUELI"]
    };
    function atualizarFuncionarios(){
      const filial = document.getElementById('filial-folgas').value;
      const sel = document.getElementById('funcionario-folgas');
      sel.innerHTML = "<option value=''>Selecione um funcionário</option>";
      (funcionariosPorFilial[filial]||[]).forEach(n=>{
        const op = document.createElement("option"); op.value=n; op.textContent=n; sel.appendChild(op);
      });
    }

    // ====== REGRAS FOLGAS ======
    document.getElementById('dataTrabalho-folgas').addEventListener('change', ()=>{
      const m = document.getElementById('motivo-folgas'); if(m.value!==''){ verificarMotivo(); }
    });
    document.getElementById('motivo-folgas').addEventListener('change', verificarMotivo);
    function verificarMotivo(){
      const motivo = document.getElementById('motivo-folgas').value;
      const dtTrabalho = document.getElementById("dataTrabalho-folgas");
      const dtFolga = document.getElementById("dataFolga-folgas");
      const outros = document.getElementById("motivoOutros-folgas");
      if(!dtTrabalho.value){ alert("Selecione primeiro o 'Dia Trabalhado'! "); document.getElementById('motivo-folgas').value=''; return; }
      const base = new Date(dtTrabalho.value+'T00:00:00'); const maxDate = new Date(base);
      if(motivo==="DOMINGO"){ maxDate.setDate(base.getDate()+7); }
      else if(motivo==="FERIADO" || motivo==="OUTROS"){ maxDate.setDate(base.getDate()+60); }
      dtFolga.min = base.toISOString().split('T')[0]; dtFolga.max = maxDate.toISOString().split('T')[0];
      outros.style.display = motivo==="OUTROS" ? "block":"none";
      document.getElementById('outrosMotivo-folgas').required = (motivo==="OUTROS");
    }
    document.getElementById("form-folgas").addEventListener("submit", function(e){
      e.preventDefault();
      fetch(this.action,{method:"POST", body:new FormData(this)})
        .then(r=>r.text()).then(()=>{
          alert("Folga cadastrada com sucesso!");
          this.reset();
          document.getElementById("funcionario-folgas").innerHTML = '<option value="">Selecione a filial primeiro</option>';
          document.getElementById("motivoOutros-folgas").style.display="none";
        }).catch(()=>alert("Erro ao enviar os dados!"));
    });

    // ====== FALTA ======
    const filialSelectFalta = document.getElementById('filial-falta');
    const funcionarioSelectFalta = document.getElementById('funcionario-falta');
    const motivoSelectFalta = document.getElementById('motivo-falta');
    const diasContainer = document.getElementById('dias-afastamento-container');
    const formFalta = document.getElementById('form-falta');
    const submitButtonFalta = formFalta.querySelector('button[type="submit"]');

    filialSelectFalta.addEventListener('change', ()=>{
      const f = filialSelectFalta.value;
      funcionarioSelectFalta.innerHTML = '<option value="">Selecione um funcionário</option>';
      (funcionariosPorFilial[f]||[]).forEach(n=>{
        const op=document.createElement('option'); op.value=n; op.textContent=n; funcionarioSelectFalta.appendChild(op);
      });
    });
    motivoSelectFalta.addEventListener('change', ()=>{ diasContainer.style.display = motivoSelectFalta.value==='ATESTADO MÉDICO' ? 'block':'none'; });
    formFalta.addEventListener('submit', e=>{
      e.preventDefault(); submitButtonFalta.disabled=true;
      const fd = new FormData(formFalta);
      const inpt = document.getElementById('data-falta'); const v = inpt.value;
      if(!v){ alert("Informe a Data da Falta."); submitButtonFalta.disabled=false; return; }
      const [yyyy,mm,dd] = v.split('-'); fd.set("data_falta", `${dd}/${mm}/${yyyy}`);
      fetch('https://script.google.com/macros/s/AKfycbxu_jVaotWytMOQh4UCZetFZFOxgk5ePrOkaviDd-qKNPiu2_8BjCaNczAVZzaDwAbj/exec', { method:'POST', body:fd })
        .then(r=>r.text()).then(()=>{
          alert("Falta cadastrada com sucesso!"); formFalta.reset(); diasContainer.style.display='none'; submitButtonFalta.disabled=false;
        }).catch(()=>{ alert("Erro ao registrar a falta. Tente novamente."); submitButtonFalta.disabled=false; });
    });

    // ====== SACOLA ======
    function toggleCamposG(){ const g=document.getElementById("grupo-tamanho-g"); const cb=document.getElementById("mostrar-g"); g.style.display= cb.checked?"block":"none"; }
    function getDataHoraAtualBR(){ const agora = new Date(); return agora.toLocaleString('pt-BR',{timeZone:'America/Sao_Paulo'}); }
    function preencherDataSacola(){ document.getElementById("data-contagem").value = getDataHoraAtualBR(); }
    document.getElementById("form-sacola").addEventListener("submit", async function(e){
      e.preventDefault();
      const cbG=document.getElementById("mostrar-g"), sG=document.getElementById("sacola-g"), eG=document.getElementById("envelope-g");
      if(cbG.checked && (sG.value===""||eG.value==="")){ alert("Preencha os campos de Sacola G e Envelope G."); return; }
      const btn=document.getElementById("btn-enviar"); btn.disabled=true;
      document.getElementById("data-contagem").value = getDataHoraAtualBR();
      try{
        await fetch("https://script.google.com/macros/s/AKfycbxu_jVaotWytMOQh4UCZetFZFOxgk5ePrOkaviDd-qKNPiu2_8BjCaNczAVZzaDwAbj/exec",{ method:"POST", body:new FormData(e.target) });
        alert("Contagem registrada com sucesso!"); e.target.reset(); toggleCamposG(); document.getElementById("data-contagem").value = getDataHoraAtualBR();
      }catch(err){ alert("Erro ao registrar os dados. Tente novamente."); console.error(err); } finally{ btn.disabled=false; }
    });

    // ====== DIVERGÊNCIA ======
    let isSubmittingDiv=false;
    document.getElementById('transportadora-div').addEventListener('change', function(){
      const outrosDiv = document.getElementById('outrosTransportadoraDiv');
      if(this.value==='OUTROS'){ outrosDiv.style.display='block'; document.getElementById('outraTransportadoraDiv').required=true; }
      else{ outrosDiv.style.display='none'; document.getElementById('outraTransportadoraDiv').required=false; }
    });
    function enviarFormularioDivergencia(event){
      event.preventDefault(); if(isSubmittingDiv) return; isSubmittingDiv=true;
      const btn = event.target.querySelector("button[type='submit']"); const load = document.getElementById("loadingMessageDiv");
      btn.disabled=true; btn.textContent="Enviando..."; load.style.display="block";
      const fd = new FormData(document.getElementById("formulario-divergencia"));
      fetch("https://script.google.com/macros/s/AKfycbw5xq6i5Qoc0s3f-ZaQ6FCZdsjXrC_my8d0tmgr756hWZQqT9Olu9DjsGOYwTlvnBQA/exec",{ method:"POST", body:fd })
        .then(r=>r.json()).then(()=>{
          alert("SUA DIVERGÊNCIA FOI ENVIADA COM SUCESSO, AGRADECEMOS SEU APOIO");
          document.getElementById('formulario-divergencia').reset();
          document.getElementById('outrosTransportadoraDiv').style.display='none';
        }).catch(()=>{ alert("Erro ao enviar o formulário. Tente novamente."); })
        .finally(()=>{ setTimeout(()=>{ btn.disabled=false; btn.textContent="Enviar"; isSubmittingDiv=false; load.style.display="none"; },100); });
    }

    // ====== TRANSFERÊNCIA ======
    function atualizarEmailTrans(){
      const origem = document.getElementById('filial-origem').value;
      const selDest = document.getElementById('filial-destino');
      const emailPorFilial = {
        ARTUR:"heringarturmachado@gmail.com", FLORIANO:"hs.uberlandia.floriano@gmail.com",
        JOTA:"brunohenzo09@gmail.com", MODA:"dlaire28@gmail.com", PONTO:"soniameiry@gmail.com",
        JA:"jaugustocoliveira@terra.com.br", JE:"jeoliveira1966@gmail.com"
      };
      document.getElementById('email-trans').value = emailPorFilial[origem]||"";
      Array.from(selDest.options).forEach(o=>{ o.disabled = (o.value===origem && o.value!==""); });
      if(selDest.value===origem){ selDest.value=""; }
    }
    function contarLinhasTrans(){
      const linhas = document.getElementById('mercadorias-trans').value.split('\n').filter(l=>l.trim()!=='');
      document.getElementById('total-itens-trans').textContent = linhas.length;
    }
    document.getElementById('transfer-form').addEventListener('submit', e=>{ e.preventDefault(); enviarFormularioTrans(); });
    function enviarFormularioTrans(){
      const form = document.getElementById('transfer-form');
      if(!form.checkValidity()){ mostrarMensagemErroTrans("Por favor, preencha todos os campos obrigatórios!"); return; }
      const data = new URLSearchParams(new FormData(form)).toString();
      document.getElementById('loading-overlay-trans').style.display='flex';
      fetch("https://script.google.com/macros/s/AKfycbxu_jVaotWytMOQh4UCZetFZFOxgk5ePrOkaviDd-qKNPiu2_8BjCaNczAVZzaDwAbj/exec",{
        method:"POST", headers:{ "Content-Type":"application/x-www-form-urlencoded" }, body:data
      })
      .then(r=>r.json()).then(res=>{
        document.getElementById('loading-overlay-trans').style.display='none';
        if(res.numeroTransferencia){ mostrarMensagemSucessoTrans(); exibirNumeroTransferencia(res.numeroTransferencia); setTimeout(limparFormularioTrans,5000); }
        else{ mostrarMensagemErroTrans("Erro ao enviar: Resposta inválida do servidor."); }
      }).catch(()=>{
        document.getElementById('loading-overlay-trans').style.display='none';
        mostrarMensagemErroTrans("Erro ao enviar o formulário. Tente novamente.");
      });
    }
    function mostrarMensagemSucessoTrans(){ document.getElementById('success-message-trans').style.display='block'; document.getElementById('error-message-trans').style.display='none'; }
    function mostrarMensagemErroTrans(m){ const e=document.getElementById('error-message-trans'); e.innerHTML=m; e.style.display='block'; document.getElementById('success-message-trans').style.display='none'; }
    function exibirNumeroTransferencia(n){ document.getElementById('numero-transferencia').style.display='block'; document.getElementById('transfer-id').textContent=n; }
    function limparFormularioTrans(){ document.getElementById('transfer-form').reset(); document.getElementById('numero-transferencia').style.display='none'; document.getElementById('total-itens-trans').textContent='0'; contarLinhasTrans(); }

    // ====== GERADOR EAN13 ======
    function calculateChecksum(code){ let sum=0; for(let i=0;i<12;i++){ const d=parseInt(code[i]); sum += (i%2===0)? d*1 : d*3; } return (10-(sum%10))%10; }
    function generateEAN13(code, skipValidation=false){
      if(!/^\d{12,13}$/.test(code)) throw new Error(`Código inválido: "${code}" - Deve conter 12 ou 13 dígitos`);
      if(code.length===12){ code += calculateChecksum(code); } else if(!skipValidation && calculateChecksum(code.substring(0,12)) != code[12]){ throw new Error(`Dígito verificador inválido para código: "${code}"`); }
      const P={ L:["0001101","0011001","0010011","0111101","0100011","0110001","0101111","0111011","0110111","0001011"], G:["0100111","0110011","0011011","0100001","0011101","0111001","0000101","0010001","0001001","0010111"], R:["1110010","1100110","1101100","1000010","1011100","1001110","1010000","1000100","1001000","1110100"] };
      const S=["LLLLLL","LLGLGG","LLGGLG","LLGGGL","LGLLGG","LGGLLG","LGGGLL","LGLGLG","LGLGGL","LGGLGL"]; const fd=parseInt(code[0]); const pat=S[fd];
      let bin="101"; for(let i=1;i<=6;i++){ const d=parseInt(code[i]); bin += P[pat[i-1]][d]; } bin+="01010"; for(let i=7;i<=12;i++){ const d=parseInt(code[i]); bin+=P.R[d]; } bin+="101";
      return { code:code, binary:bin };
    }
    function renderBarcode(data, container){
      const bin=data.binary, code=data.code, w=2, h=60, m=10, f=14;
      const svg=document.createElementNS("http://www.w3.org/2000/svg","svg");
      const W=(bin.length*w)+(m*2), H=h+f+15; svg.setAttribute("width",W); svg.setAttribute("height",H); svg.setAttribute("class","barcode-svg");
      let x=m; for(let i=0;i<bin.length;i++){ if(bin[i]==='1'){ const r=document.createElementNS("http://www.w3.org/2000/svg","rect"); r.setAttribute("x",x); r.setAttribute("y",m); r.setAttribute("width",w); r.setAttribute("height",h); r.setAttribute("fill","#000"); svg.appendChild(r);} x+=w; }
      const t=document.createElementNS("http://www.w3.org/2000/svg","text"); t.setAttribute("x",W/2); t.setAttribute("y",H-5); t.setAttribute("text-anchor","middle"); t.setAttribute("font-size",f); t.setAttribute("font-family","monospace"); t.setAttribute("fill","white"); t.textContent=code; svg.appendChild(t);
      const item=document.createElement("div"); item.className="barcode-item"; item.appendChild(svg); container.appendChild(item);
    }
    function gerarTodosBarras(){
      const input=document.getElementById("codigos-barras").value.trim(); const container=document.getElementById("barcodes"); container.innerHTML='';
      if(!input){ alert("Por favor, cole alguns códigos EAN13 no campo de texto."); return; }
      const codigos=input.split('\n').map(l=>l.trim()).filter(l=>l.length>0); let ok=0, err=0; const msgs=[];
      codigos.forEach((c,i)=>{ try{ const b=generateEAN13(c,true); renderBarcode(b,container); ok++; } catch(e){ err++; msgs.push(`Linha ${i+1}: ${e.message}`); const d=document.createElement("div"); d.className="barcode-item no-print"; d.style.color="red"; d.textContent=`Erro: ${c} - ${e.message}`; container.appendChild(d); } });
      if(err>0){ alert(`Foram gerados ${ok} códigos com sucesso.\n\nErros encontrados (${err}):\n${msgs.join('\n')}`); } else if(ok>0){ alert(`Todos os ${ok} códigos foram gerados com sucesso!`); }
    }
    function limparTudoBarras(){ document.getElementById("codigos-barras").value=''; document.getElementById("barcodes").innerHTML=''; }
    function copiarCodigosBarras(){ const i=document.getElementById("codigos-barras"); i.select(); document.execCommand('copy'); alert("Códigos copiados para a área de transferência!"); }

    // ====== NF-e (Login & Histórico) ======
    const URL_SCRIPT = "https://script.google.com/macros/s/AKfycbwfoYOgleHUcmbr_1B8tV_NG6cEZxcHm5zBSrJ0ItgRV_Cp7tumh3GjBzsvzTSNJ5sbmA/exec";
    const filiaisValidas = ["293","488","287","288","761"];
    let scannerInstance = null;
    let torchAvailable = false;

    function entrarNF(){
      const codigo=document.getElementById('codigo-nf').value.trim();
      if(!codigo || !filiaisValidas.includes(codigo)){ alert("Código de filial inválido."); return; }
      localStorage.setItem('filial',codigo);
      document.getElementById('login-nf').classList.add('hidden');
      document.getElementById('principal-nf').classList.remove('hidden');
      carregarHistoricoNF(codigo);
    }
    function sairNF(){
      localStorage.removeItem('filial');
      document.getElementById('login-nf').classList.remove('hidden');
      document.getElementById('principal-nf').classList.add('hidden');
      document.getElementById('codigo-nf').value='';
    }
    function limparHistoricoLocalNF(){
      const senha=prompt("Digite a senha para limpar o histórico:"); if(senha!=="hs"){ alert("Senha incorreta. Operação cancelada."); return; }
      const filial=localStorage.getItem('filial'); localStorage.removeItem(`historico_${filial}`);
      document.getElementById('historicoLista-nf').innerHTML=''; alert("Histórico local apagado.");
    }

    function consultarNotaNF(){
      const filial=localStorage.getItem('filial'); const inpt=document.getElementById('chave-nf'); const chave=inpt.value.trim();
      const resultado=document.getElementById('resultado-nf'); const erro=document.getElementById('erro-nf'); const loading=document.getElementById('loading-nf');
      resultado.classList.add('hidden'); erro.innerText='';
      if(!filial || chave.length!==44){ erro.innerText='Preencha corretamente a chave com 44 dígitos.'; return; }
      const hist=JSON.parse(localStorage.getItem(`historico_${filial}`))||[]; if(hist.some(h=>h.chave===chave)){ erro.innerText='Essa chave já foi consultada anteriormente!'; return; }
      loading.classList.remove('hidden');
      fetch(`${URL_SCRIPT}?chave=${chave}&filial=${filial}`).then(res=>res.json()).then(data=>{
        if(data.success){
          resultado.classList.remove('hidden');
          resultado.innerHTML = `<p><strong>Número da NF:</strong> ${data.data.numeroNF}</p>
                                 <p><strong>Valor Total:</strong> ${data.data.valorTotal}</p>
                                 <p><strong>Quantidade Total:</strong> ${data.data.quantidadeTotal}</p>
                                 <p><strong>Status:</strong> ✅ ${data.data.status}</p>`;
          hist.push({ chave, dataHora:new Date().toISOString(), ...data.data });
          localStorage.setItem(`historico_${filial}`, JSON.stringify(hist));
          carregarHistoricoNF(filial);
        }else{ erro.innerText = data.message || 'Erro ao buscar nota fiscal.'; }
      }).catch(()=> erro.innerText='Erro de comunicação com o servidor.')
        .finally(()=>{ loading.classList.add('hidden'); inpt.value=''; });
    }
    function carregarHistoricoNF(filial){
      const lista=document.getElementById('historicoLista-nf'); const hist=JSON.parse(localStorage.getItem(`historico_${filial}`))||[];
      lista.innerHTML=''; if(hist.length===0){ lista.innerHTML='<li>Nenhum histórico local encontrado.</li>'; return; }
      hist.slice().reverse().forEach(it=>{ const d=new Date(it.dataHora).toLocaleDateString('pt-BR'); lista.innerHTML += `<li>${d} - NF ${it.numeroNF} - ${it.valorTotal} - ${it.quantidadeTotal} itens</li>`; });
    }

    // ====== SCANNER DYNAMSOFT MELHORADO ======
    function extractChaveFromText(txt){
      if(/^\d{44}$/.test(txt)) return txt;
      const m = txt.match(/[?&]chNFe=([0-9]{44})/i);
      return m ? m[1] : null;
    }

    async function abrirLeitorNF(){
      const campo = document.getElementById("chave-nf");
      if(campo.value.trim()!==''){ alert("Limpe o campo antes de escanear outro código."); return; }

      const wrapper = document.getElementById("scanner");
      const videoHost = document.getElementById("scanner-video");
      const torchBtn = document.getElementById("torch-btn");

      try{
        // Se já existir um scanner antigo, mata o contexto
        if(scannerInstance){
          try{ await scannerInstance.hide(); await scannerInstance.stop(); await scannerInstance.destroyContext(); }catch(_){}
          scannerInstance = null;
        }

        wrapper.style.display="flex";
        scannerInstance = await Dynamsoft.DBR.BarcodeScanner.createInstance();

        // 1) UI no container dedicado (mantém vídeo preso ao nosso div)
        await scannerInstance.setUIElement(videoHost);

        // 2) Região central (ajuda a “mirar” e foca no meio)
        await scannerInstance.setScanRegion({
          regionLeft: 15, regionTop: 15, regionRight: 85, regionBottom: 85,
          regionMeasuredByPercentage: 1
        });

        // 3) Presets + formatos focados (QR para NF-e, plus alguns 1D comuns)
        await scannerInstance.updateRuntimeSettings("speed");
        try{
          const rs = await scannerInstance.getRuntimeSettings();
          // Habilita apenas os formatos mais relevantes para nota/chave
          rs.barcodeFormatIds =
            Dynamsoft.DBR.EnumBarcodeFormat.BF_QR_CODE |
            Dynamsoft.DBR.EnumBarcodeFormat.BF_CODE_128 |
            Dynamsoft.DBR.EnumBarcodeFormat.BF_EAN_13 |
            Dynamsoft.DBR.EnumBarcodeFormat.BF_EAN_8;
          await scannerInstance.updateRuntimeSettings(rs);
        }catch(_){ /* algumas builds não expõem getRuntimeSettings; segue com speed preset */ }

        // 4) Configs de leitura (evita repetição)
        try{
          await scannerInstance.setScanSettings({
            duplicateForgetTime: 1500, // ms
            intervalTime: 30
          });
        }catch(_){}

        // 5) Preferência de câmera/resolução (nem todos navegadores aceitam tudo)
        try{
          await scannerInstance.setVideoSettings({
            video: {
              facingMode: { ideal: "environment" },
              width: { ideal: 1280 }, height: { ideal: 720 }
            }
          });
        }catch(_){}

        // 6) Eventos
        scannerInstance.onFrameRead = results => {
          for(const r of results){
            const txt = r.barcodeText || "";
            const chave = extractChaveFromText(txt);
            if(chave){
              campo.value = chave;
              fecharLeitorNF(true);
              break;
            }
          }
        };

        // 7) Checagem de lanterna
        torchAvailable = false;
        try{
          const cam = await scannerInstance.getCurrentCamera();
          if(cam && cam.torchFeature === true){ torchAvailable = true; }
        }catch(_){}
        torchBtn.classList.toggle('hidden', !torchAvailable);

        // 8) Exibir scanner
        await scannerInstance.show();

      }catch(ex){
        alert("Erro ao iniciar o leitor: " + ex.message);
        wrapper.style.display="none";
      }
    }

    async function toggleTorchNF(){
      if(!scannerInstance) return;
      try{
        const enabled = await scannerInstance.isTorchOn();
        await scannerInstance.toggleTorch(!enabled);
      }catch(_){}
    }

    async function fecharLeitorNF(foiLeitura=false){
      const wrapper = document.getElementById("scanner");
      try{
        if(scannerInstance){
          await scannerInstance.hide();
          await scannerInstance.stop();
          await scannerInstance.destroyContext();
        }
      }catch(_){}
      scannerInstance = null;
      wrapper.style.display="none";

      // Se foi leitura, rola a tela de volta pro campo e foca nele
      if(foiLeitura){
        document.getElementById('chave-nf').scrollIntoView({ behavior:'smooth', block:'center' });
        document.getElementById('chave-nf').focus();
      }
    }

    // Restaura sessão NF se já logado
    window.addEventListener('load', function(){
      const filial=localStorage.getItem('filial');
      if(filial){
        document.getElementById('login-nf').classList.add('hidden');
        document.getElementById('principal-nf').classList.remove('hidden');
        carregarHistoricoNF(filial);
      }
    });

    // Preenche data de sacola quando a seção abre
    document.addEventListener('DOMContentLoaded', function(){
      const observer=new MutationObserver(function(mutations){
        mutations.forEach(function(m){
          if(m.type==='attributes' && m.attributeName==='class'){
            const t=m.target;
            if(t.id==='sacola' && t.classList.contains('active')){ preencherDataSacola(); }
          }
        });
      });
      document.querySelectorAll('.section').forEach(s=>{
        observer.observe(s,{ attributes:true });
      });
    });
  </script>
</body>
</html>
