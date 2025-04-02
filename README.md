<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta de NF - Hering</title>
  <style>
    :root {
      --cor-primaria: #4285f4;
      --cor-sucesso: #34a853;
      --cor-erro: #ea4335;
      --cor-aviso: #fbbc05;
    }
    
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Roboto', Arial, sans-serif;
    }
    
    body {
      background-color: #f8f9fa;
      color: #202124;
      line-height: 1.6;
      padding: 20px;
      max-width: 100%;
    }
    
    .container {
      background: white;
      border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
      margin: 0 auto;
      max-width: 600px;
      overflow: hidden;
      padding: 30px;
    }
    
    h1 {
      color: var(--cor-primaria);
      font-size: 24px;
      margin-bottom: 25px;
      text-align: center;
    }
    
    .form-group {
      margin-bottom: 20px;
    }
    
    label {
      display: block;
      font-size: 14px;
      font-weight: 500;
      margin-bottom: 8px;
    }
    
    select, input {
      background-color: #f8f9fa;
      border: 1px solid #dadce0;
      border-radius: 4px;
      font-size: 16px;
      padding: 12px 15px;
      width: 100%;
      transition: border-color 0.3s;
    }
    
    select:focus, input:focus {
      border-color: var(--cor-primaria);
      outline: none;
    }
    
    button {
      background-color: var(--cor-primaria);
      border: none;
      border-radius: 4px;
      color: white;
      cursor: pointer;
      font-size: 16px;
      font-weight: 500;
      margin-top: 10px;
      padding: 14px;
      transition: background-color 0.3s;
      width: 100%;
    }
    
    button:hover {
      background-color: #3367d6;
    }
    
    button:disabled {
      background-color: #b8c2cc;
      cursor: not-allowed;
    }
    
    #loading {
      align-items: center;
      display: none;
      flex-direction: column;
      justify-content: center;
      margin: 20px 0;
      text-align: center;
    }
    
    .spinner {
      animation: spin 1s linear infinite;
      border: 3px solid rgba(0, 0, 0, 0.1);
      border-radius: 50%;
      border-top-color: var(--cor-primaria);
      height: 30px;
      margin-bottom: 10px;
      width: 30px;
    }
    
    @keyframes spin {
      to { transform: rotate(360deg); }
    }
    
    #resultado {
      border-left: 4px solid transparent;
      border-radius: 4px;
      display: none;
      margin-top: 25px;
      padding: 20px;
    }
    
    .success {
      background-color: #e8f8f5;
      border-left-color: var(--cor-sucesso);
    }
    
    .error {
      background-color: #fdedec;
      border-left-color: var(--cor-erro);
    }
    
    .warning {
      background-color: #fff8e6;
      border-left-color: var(--cor-aviso);
    }
    
    .result-title {
      font-size: 18px;
      margin-bottom: 15px;
    }
    
    .result-detail {
      margin-bottom: 8px;
    }
    
    .status {
      border-radius: 4px;
      display: inline-block;
      font-weight: 500;
      padding: 2px 8px;
    }
    
    .status-received {
      background-color: #c8e6c9;
      color: #388e3c;
    }
    
    .status-pending {
      background-color: #ffe0b2;
      color: #e65100;
    }
    
    .footer {
      color: #5f6368;
      font-size: 12px;
      margin-top: 30px;
      text-align: center;
    }
    
    @media (max-width: 480px) {
      .container {
        padding: 20px;
      }
      
      h1 {
        font-size: 20px;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Consulta de Notas Fiscais</h1>
    
    <form id="consultaForm">
      <div class="form-group">
        <label for="filial">Filial:</label>
        <select id="filial" required>
          <option value="">Selecione a filial</option>
          <option value="ARTUR">ARTUR</option>
          <option value="FLORIANO">FLORIANO</option>
          <option value="JOTA">JOTA</option>
          <option value="MODA">MODA</option>
          <option value="PONTO">PONTO</option>
        </select>
      </div>
      
      <div class="form-group">
        <label for="numeroNF">Número da Nota Fiscal:</label>
        <input type="text" id="numeroNF" required placeholder="Digite o número completo">
      </div>
      
      <button type="submit" id="btnConsultar">
        <span id="btnText">CONSULTAR</span>
      </button>
    </form>
    
    <div id="loading">
      <div class="spinner"></div>
      <p>Consultando banco de dados...</p>
    </div>
    
    <div id="resultado"></div>
    
    <div class="footer">
      Sistema de Consulta de NF - Hering &copy; 2023
    </div>
  </div>

  <script>
    // Verificação de ambiente
    if (typeof google === 'undefined') {
      document.getElementById('consultaForm').style.display = 'none';
      document.getElementById('resultado').innerHTML = `
        <div class="error">
          <h3 class="result-title">Erro de Carregamento</h3>
          <p>Esta página deve ser acessada através do link oficial do sistema.</p>
          <p><strong>Como resolver:</strong></p>
          <ol>
            <li>Acesse via URL do Google Apps Script (termina em /exec)</li>
            <li>Não abra o arquivo HTML diretamente no navegador</li>
            <li>Se você é o administrador, publique corretamente o aplicativo</li>
          </ol>
        </div>
      `;
      document.getElementById('resultado').style.display = 'block';
    } else {
      // Configuração quando tudo estiver OK
      document.addEventListener('DOMContentLoaded', function() {
        const form = document.getElementById('consultaForm');
        const btnConsultar = document.getElementById('btnConsultar');
        const btnText = document.getElementById('btnText');
        const loadingDiv = document.getElementById('loading');
        const resultadoDiv = document.getElementById('resultado');
        
        form.addEventListener('submit', function(e) {
          e.preventDefault();
          
          const filial = document.getElementById('filial').value;
          const numeroNF = document.getElementById('numeroNF').value.trim();
          
          // Validação básica
          if (!filial || !numeroNF) {
            resultadoDiv.innerHTML = `
              <div class="warning">
                <h3 class="result-title">Atenção</h3>
                <p>Preencha todos os campos corretamente.</p>
              </div>
            `;
            resultadoDiv.style.display = 'block';
            return;
          }
          
          // Mostra loading
          resultadoDiv.style.display = 'none';
          loadingDiv.style.display = 'flex';
          btnConsultar.disabled = true;
          btnText.textContent = 'PROCESSANDO...';
          
          // Chamada para o Google Script
          google.script.run
            .withSuccessHandler(function(res) {
              loadingDiv.style.display = 'none';
              btnConsultar.disabled = false;
              btnText.textContent = 'CONSULTAR';
              
              resultadoDiv.innerHTML = '';
              resultadoDiv.className = '';
              
              if (res.success) {
                if (res.encontrada) {
                  resultadoDiv.innerHTML = `
                    <h3 class="result-title">Nota Fiscal Encontrada</h3>
                    <p class="result-detail"><strong>Filial:</strong> ${res.dados.filial}</p>
                    <p class="result-detail"><strong>Número NF:</strong> ${res.dados.numeroNF}</p>
                    <p class="result-detail"><strong>Data Recebimento:</strong> ${res.dados.dataRecebimento || 'Não informada'}</p>
                    <p class="result-detail"><strong>Status:</strong> 
                      <span class="status ${res.dados.status === 'RECEBIDA' ? 'status-received' : 'status-pending'}">
                        ${res.dados.status}
                      </span>
                    </p>
                  `;
                  resultadoDiv.className = 'success';
                } else {
                  resultadoDiv.innerHTML = `
                    <h3 class="result-title">Nota Fiscal Não Encontrada</h3>
                    <p>A nota fiscal <strong>${numeroNF}</strong> não foi encontrada na filial <strong>${filial}</strong>.</p>
                    <p>Verifique o número ou consulte o departamento responsável.</p>
                  `;
                  resultadoDiv.className = 'warning';
                }
              } else {
                resultadoDiv.innerHTML = `
                  <h3 class="result-title">Erro no Sistema</h3>
                  <p>${res.message || 'Erro desconhecido'}</p>
                  <p>Tente novamente mais tarde.</p>
                `;
                resultadoDiv.className = 'error';
              }
              
              resultadoDiv.style.display = 'block';
            })
            .withFailureHandler(function(error) {
              loadingDiv.style.display = 'none';
              btnConsultar.disabled = false;
              btnText.textContent = 'CONSULTAR';
              
              resultadoDiv.innerHTML = `
                <h3 class="result-title">Erro de Comunicação</h3>
                <p>${error.message || 'Falha ao conectar com o servidor'}</p>
                <p>Por favor, tente novamente.</p>
              `;
              resultadoDiv.className = 'error';
              resultadoDiv.style.display = 'block';
              
              console.error("Erro detalhado:", error);
            })
            .consultarNF(filial, numeroNF);
        });
      });
    }
  </script>
</body>
</html>
