<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta NF</title>
  <style>
    body { font-family: Arial; max-width: 500px; margin: 0 auto; padding: 20px; }
    form { background: #f5f5f5; padding: 20px; border-radius: 5px; }
    input, select { width: 100%; padding: 10px; margin: 5px 0; }
    button { background: #4285f4; color: white; border: none; padding: 10px; width: 100%; }
    #resultado { margin-top: 20px; padding: 15px; border-radius: 5px; }
    .ok { background: #e8f5e9; }
    .erro { background: #ffebee; }
  </style>
</head>
<body>
  <h1>Consulta de NF</h1>
  
  <form id="formConsulta">
    <select id="filial" required>
      <option value="">Selecione a filial</option>
      <option value="ARTUR">ARTUR</option>
      <option value="FLORIANO">FLORIANO</option>
    </select>
    
    <input type="text" id="numeroNF" placeholder="Número da NF" required>
    
    <button type="submit">Consultar</button>
  </form>
  
  <div id="resultado"></div>

  <script>
    document.getElementById('formConsulta').addEventListener('submit', function(e) {
      e.preventDefault();
      
      var filial = document.getElementById('filial').value;
      var numeroNF = document.getElementById('numeroNF').value;
      var resultadoDiv = document.getElementById('resultado');
      
      resultadoDiv.innerHTML = "Consultando...";
      resultadoDiv.className = "";
      
      google.script.run
        .withSuccessHandler(function(res) {
          if (res.erro) {
            resultadoDiv.innerHTML = res.erro;
            resultadoDiv.className = "erro";
          } else if (res.encontrada) {
            resultadoDiv.innerHTML = `
              <h3>NF Encontrada</h3>
              <p><strong>Filial:</strong> ${res.filial}</p>
              <p><strong>NF:</strong> ${res.numeroNF}</p>
              <p><strong>Data:</strong> ${res.data}</p>
              <p><strong>Status:</strong> ${res.status}</p>
            `;
            resultadoDiv.className = "ok";
          } else {
            resultadoDiv.innerHTML = "NF não encontrada!";
            resultadoDiv.className = "erro";
          }
        })
        .withFailureHandler(function(err) {
          resultadoDiv.innerHTML = "Erro: " + err.message;
          resultadoDiv.className = "erro";
        })
        .consultarNF(filial, numeroNF);
    });
  </script>
</body>
</html>
