<html>
<head>
  <base target="_top">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta de Notas Fiscais</title>
  <style>
    * {
      box-sizing: border-box;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }
    body {
      max-width: 800px;
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
      font-size: 24px;
    }
    .form-group {
      margin-bottom: 20px;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: 600;
      color: #34495e;
      font-size: 14px;
    }
    select, input {
      width: 100%;
      padding: 12px 15px;
      border: 1px solid #ddd;
      border-radius: 6px;
      font-size: 16px;
      background-color: #f8f9fa;
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
    .nf-encontrada {
      background-color: #e8f5e9;
      border-left: 4px solid #4caf50;
    }
    .nf-pendente {
      background-color: #fff3e0;
      border-left: 4px solid #ff9800;
    }
    .nf-nao-encontrada {
      background-color: #ffebee;
      border-left: 4px solid #f44336;
    }
    .resultado-titulo {
      margin-top: 0;
      margin-bottom: 15px;
      font-size: 18px;
    }
    .detalhes-nf p {
      margin: 8px 0;
      line-height: 1.6;
    }
    .status {
      font-weight: 600;
      padding: 3px 8px;
      border-radius: 4px;
      font-size: 14px;
    }
    .recebida {
      color: #388e3c;
      background-color: #c8e6c9;
    }
    .pendente {
      color: #e65100;
      background-color: #ffe0b2;
    }
    .loading {
      display: inline-block;
      width: 20px;
      height: 20px;
      border: 3px solid rgba(255,255,255,.3);
      border-radius: 50%;
      border-top-color: white;
      animation: spin 1s ease-in-out infinite;
      margin-right: 10px;
      vertical-align: middle;
    }
    @keyframes spin {
      to { transform: rotate(360deg); }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>CONSULTA DE NOTAS FISCAIS</h1>
    
    <form id="consultaForm">
      <div class="form-group">
        <label for="filial">FILIAL:</label>
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
        <label for="numeroNF">NÚMERO DA NOTA FISCAL:</label>
        <input type="text" id="numeroNF" required placeholder="Digite o número completo">
      </div>
      
      <button type="submit" id="btnConsultar">
        <span id="btnText">CONSULTAR</span>
      </button>
    </form>
    
    <div id="resultado">
      <h3 class="resultado-titulo" id="resultadoTitulo"></h3>
      <div id="detalhesNF" class="detalhes-nf"></div>
    </div>
  </div>

  <script>
    document.addEventListener('DOMContentLoaded', function() {
      const form = document.getElementById('consultaForm');
      const btnConsultar = document.getElementById('btnConsultar');
      const btnText = document.getElementById('btnText');
      
      form.addEventListener('submit', function(e) {
        e.preventDefault();
        
        const filial = document.getElementById('filial').value;
        const numeroNF = document.getElementById('numeroNF').value.trim();
        
        if (!filial || !numeroNF) return;
        
        // Mostra loading
        btnConsultar.disabled = true;
        btnText.innerHTML = '<span class="loading"></span> PROCESSANDO...';
        
        // Chama o Web App
        google.script.run
          .withSuccessHandler(handleSuccess)
          .withFailureHandler(handleError)
          .consultarNF(filial, numeroNF);
      });
      
      function handleSuccess(result) {
        const resultadoDiv = document.getElementById('resultado');
        const titulo = document.getElementById('resultadoTitulo');
        const detalhes = document.getElementById('detalhesNF');
        
        // Resetar estilos
        resultadoDiv.className = '';
        resultadoDiv.style.display = 'block';
        
        if (result.encontrada) {
          resultadoDiv.classList.add(result.dados.status === 'RECEBIDA' ? 'nf-encontrada' : 'nf-pendente');
          
          titulo.textContent = `NOTA FISCAL ${result.dados.status === 'RECEBIDA' ? 'ENCONTRADA' : 'PENDENTE'}`;
          
          let html = `
            <p><strong>Filial:</strong> ${filial.value}</p>
            <p><strong>Número NF:</strong> ${numeroNF.value}</p>
            <p><strong>Status:</strong> <span class="status ${result.dados.status === 'RECEBIDA' ? 'recebida' : 'pendente'}">
              ${result.dados.status}
            </span></p>
          `;
          
          if (result.dados.dataRecebimento) {
            html += `<p><strong>Data Recebimento:</strong> ${result.dados.dataRecebimento}</p>`;
          }
          
          detalhes.innerHTML = html;
        } else {
          resultadoDiv.classList.add('nf-nao-encontrada');
          titulo.textContent = 'NOTA FISCAL NÃO ENCONTRADA';
          detalhes.innerHTML = `
            <p>A nota fiscal <strong>${numeroNF.value}</strong> não foi encontrada na filial <strong>${filial.value}</strong>.</p>
            <p>Verifique os dados informados ou consulte o departamento responsável.</p>
          `;
        }
        
        resetButton();
      }
      
      function handleError(error) {
        alert('Erro na consulta: ' + error.message);
        resetButton();
      }
      
      function resetButton() {
        btnConsultar.disabled = false;
        btnText.textContent = 'CONSULTAR';
      }
    });
  </script>
</body>
</html>
