<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta de Notas Fiscais - Hering</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      max-width: 600px;
      margin: 0 auto;
      padding: 20px;
      background-color: #f5f7fa;
    }
    .container {
      background: white;
      padding: 25px;
      border-radius: 10px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    h1 {
      color: #2c3e50;
      text-align: center;
      margin-bottom: 25px;
    }
    .form-group {
      margin-bottom: 20px;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: 600;
      color: #34495e;
    }
    select, input {
      width: 100%;
      padding: 12px 15px;
      border: 1px solid #ddd;
      border-radius: 6px;
      font-size: 16px;
    }
    button {
      background-color: #4285f4;
      color: white;
      padding: 14px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 16px;
      width: 100%;
      font-weight: 600;
      transition: background-color 0.3s;
      margin-top: 10px;
    }
    button:hover {
      background-color: #3367d6;
    }
    button:disabled {
      background-color: #b8c2cc;
      cursor: not-allowed;
    }
    #loading {
      display: none;
      text-align: center;
      padding: 20px;
      color: #555;
    }
    #resultado {
      margin-top: 25px;
      padding: 20px;
      border-radius: 8px;
      display: none;
      animation: fadeIn 0.5s;
    }
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }
    .success {
      background-color: #e8f5e9;
      border-left: 4px solid #34a853;
    }
    .error {
      background-color: #fce8e6;
      border-left: 4px solid #ea4335;
    }
    .status {
      font-weight: 600;
      padding: 3px 8px;
      border-radius: 4px;
      display: inline-block;
    }
    .received {
      background-color: #c8e6c9;
      color: #388e3c;
    }
    .pending {
      background-color: #ffe0b2;
      color: #e65100;
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
      <p>Consultando banco de dados...</p>
    </div>
    
    <div id="resultado"></div>
  </div>

  <script>
    // Verificação de ambiente - Google Script disponível?
    if (typeof google === 'undefined') {
      document.getElementById('consultaForm').style.display = 'none';
      document.getElementById('resultado').innerHTML = `
        <div class="error">
          <h3>Erro de Carregamento</h3>
          <p>Esta página deve ser acessada através do link do Google Apps Script.</p>
          <p><strong>Como resolver:</strong></p>
          <ol>
            <li>Publicar como aplicativo web no Google Script</li>
            <li>Acessar via URL gerada (terminando em /exec)</li>
            <li>Não abrir o HTML diretamente no navegador</li>
          </ol>
        </div>
      `;
      document.getElementById('resultado').style.display = 'block';
    } else {
      // Configuração do formulário quando tudo estiver OK
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
          
          // Mostra loading
          resultadoDiv.style.display = 'none';
          loadingDiv.style.display = 'block';
          btnConsultar.disabled = true;
          btnText.innerHTML = 'PROCESSANDO...';
          
          // Chamada segura para o Google Script
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
                    <h3>Nota Fiscal Encontrada</h3>
                    <p><strong>Filial:</strong> ${filial}</p>
                    <p><strong>Número NF:</strong> ${numeroNF}</p>
                    <p><strong>Data Recebimento:</strong> ${res.data || 'Não informada'}</p>
                    <p><strong>Status:</strong> <span class="status ${res.status === 'RECEBIDA' ? 'received' : 'pending'}">
                      ${res.status || 'Pendente'}
                    </span></p>
                  `;
                  resultadoDiv.className = 'success';
                } else {
                  resultadoDiv.innerHTML = `
                    <h3>Nota Fiscal Não Encontrada</h3>
                    <p>A nota fiscal <strong>${numeroNF}</strong> não foi encontrada na filial <strong>${filial}</strong>.</p>
                    <p>Verifique o número ou consulte o responsável.</p>
                  `;
                  resultadoDiv.className = 'error';
                }
              } else {
                resultadoDiv.innerHTML = `
                  <h3>Erro no Sistema</h3>
                  <p>${res.message || 'Erro desconhecido'}</p>
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
                <h3>Erro de Comunicação</h3>
                <p>${error.message || 'Falha ao conectar com o servidor'}</p>
                <p>Tente novamente mais tarde.</p>
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
