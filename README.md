<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta Rápida de NF</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 500px;
      margin: 0 auto;
      padding: 20px;
      background-color: #f5f7fa;
    }
    .container {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    select, input {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    button {
      background-color: #4285f4;
      color: white;
      border: none;
      padding: 12px;
      width: 100%;
      border-radius: 4px;
      cursor: pointer;
      font-weight: bold;
      margin-top: 10px;
    }
    #loading {
      display: none;
      text-align: center;
      padding: 20px;
      color: #555;
    }
    #resultado {
      margin-top: 20px;
      padding: 15px;
      border-radius: 4px;
      display: none;
    }
    .success {
      background-color: #e8f5e9;
      border-left: 4px solid #34a853;
    }
    .error {
      background-color: #fce8e6;
      border-left: 4px solid #ea4335;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>Consulta de Nota Fiscal</h2>
    
    <form id="formNF">
      <div>
        <label>Filial:</label>
        <select id="filial" required>
          <option value="">Selecione...</option>
          <option value="ARTUR">ARTUR</option>
          <option value="FLORIANO">FLORIANO</option>
          <option value="JOTA">JOTA</option>
          <option value="MODA">MODA</option>
          <option value="PONTO">PONTO</option>
        </select>
      </div>
      
      <div>
        <label>Número da NF:</label>
        <input type="text" id="numeroNF" required placeholder="Digite o número completo">
      </div>
      
      <button type="submit" id="btnConsultar">CONSULTAR</button>
    </form>
    
    <div id="loading">Consultando banco de dados...</div>
    
    <div id="resultado"></div>
  </div>

  <script>
    document.getElementById('formNF').addEventListener('submit', function(e) {
      e.preventDefault();
      
      const filial = document.getElementById('filial').value;
      const numeroNF = document.getElementById('numeroNF').value;
      const resultadoDiv = document.getElementById('resultado');
      const loadingDiv = document.getElementById('loading');
      const btn = document.getElementById('btnConsultar');
      
      // Reset
      resultadoDiv.style.display = 'none';
      loadingDiv.style.display = 'block';
      btn.disabled = true;
      
      // Chamada OTIMIZADA
      google.script.run
        .withSuccessHandler(function(res) {
          loadingDiv.style.display = 'none';
          btn.disabled = false;
          
          resultadoDiv.innerHTML = '';
          resultadoDiv.className = '';
          
          if (!res.success) {
            resultadoDiv.innerHTML = `<p>Erro: ${res.message}</p>`;
            resultadoDiv.className = 'error';
          } else if (res.encontrada) {
            resultadoDiv.innerHTML = `
              <h3>NF Encontrada</h3>
              <p><strong>Filial:</strong> ${res.filial}</p>
              <p><strong>Número:</strong> ${res.numeroNF}</p>
              <p><strong>Data:</strong> ${res.data}</p>
              <p><strong>Status:</strong> ${res.status}</p>
            `;
            resultadoDiv.className = 'success';
          } else {
            resultadoDiv.innerHTML = `<p>${res.message}</p>`;
            resultadoDiv.className = 'error';
          }
          
          resultadoDiv.style.display = 'block';
        })
        .withFailureHandler(function(err) {
          loadingDiv.style.display = 'none';
          btn.disabled = false;
          
          resultadoDiv.innerHTML = `<p>Erro: ${err.message}</p>`;
          resultadoDiv.className = 'error';
          resultadoDiv.style.display = 'block';
          
          console.error("Erro frontend:", err);
        })
        .consultarNF(filial, numeroNF);
    });
  </script>
</body>
</html>
