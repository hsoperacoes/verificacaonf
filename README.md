<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta de NF - Sistema Hering</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 600px;
      margin: 0 auto;
      padding: 20px;
      background-color: #f5f7fa;
    }
    .container {
      background: white;
      padding: 25px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    h1 {
      color: #2c3e50;
      text-align: center;
    }
    .form-group {
      margin-bottom: 15px;
    }
    label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    select, input {
      width: 100%;
      padding: 10px;
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
      margin-top: 10px;
    }
    #resultado {
      margin-top: 20px;
      padding: 15px;
      border-radius: 4px;
      display: none;
    }
    .loading {
      text-align: center;
      padding: 20px;
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
          <option value="">Selecione</option>
          <option value="ARTUR">ARTUR</option>
          <option value="FLORIANO">FLORIANO</option>
          <option value="JOTA">JOTA</option>
          <option value="MODA">MODA</option>
          <option value="PONTO">PONTO</option>
        </select>
      </div>
      
      <div class="form-group">
        <label for="numeroNF">Número da NF:</label>
        <input type="text" id="numeroNF" required>
      </div>
      
      <button type="submit" id="btnConsultar">Consultar</button>
    </form>
    
    <div id="resultado"></div>
  </div>

  <script>
    document.getElementById('consultaForm').addEventListener('submit', function(e) {
      e.preventDefault();
      
      var filial = document.getElementById('filial').value;
      var numeroNF = document.getElementById('numeroNF').value;
      var resultadoDiv = document.getElementById('resultado');
      var btn = document.getElementById('btnConsultar');
      
      // Mostra loading
      resultadoDiv.innerHTML = '<div class="loading">Consultando...</div>';
      resultadoDiv.style.display = 'block';
      btn.disabled = true;
      
      google.script.run
        .withSuccessHandler(function(result) {
          if (result.encontrada) {
            resultadoDiv.innerHTML = `
              <h3>NF Encontrada</h3>
              <p><strong>Status:</strong> ${result.dados.status}</p>
              ${result.dados.dataRecebimento ? 
                `<p><strong>Data:</strong> ${result.dados.dataRecebimento}</p>` : ''}
            `;
            resultadoDiv.style.backgroundColor = '#e8f5e9';
          } else {
            resultadoDiv.innerHTML = `
              <h3>NF Não Encontrada</h3>
              <p>A nota fiscal ${numeroNF} não consta no sistema.</p>
            `;
            resultadoDiv.style.backgroundColor = '#ffebee';
          }
          btn.disabled = false;
        })
        .withFailureHandler(function(error) {
          resultadoDiv.innerHTML = `
            <h3>Erro</h3>
            <p>${error.message}</p>
          `;
          resultadoDiv.style.backgroundColor = '#fff3e0';
          btn.disabled = false;
        })
        .consultarNF(filial, numeroNF);
    });
  </script>
</body>
</html>
