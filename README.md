<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta de NF - Hering</title>
  <style>
    /* (Mantenha seus estilos existentes) */
  </style>
</head>
<body>
  <div class="container">
    <h1>Consulta de Notas Fiscais</h1>
    
    <form id="consultaForm">
      <div class="form-group">
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
      
      <div class="form-group">
        <label>Número da NF:</label>
        <input type="text" id="numeroNF" required placeholder="Digite o número completo">
      </div>
      
      <button type="submit" id="btnConsultar">CONSULTAR</button>
    </form>
    
    <div id="loading">Consultando...</div>
    
    <div id="resultado"></div>
  </div>

  <script>
    document.getElementById('consultaForm').addEventListener('submit', function(e) {
      e.preventDefault();
      
      const filial = document.getElementById('filial').value;
      const numeroNF = document.getElementById('numeroNF').value;
      const resultadoDiv = document.getElementById('resultado');
      const loadingDiv = document.getElementById('loading');
      const btn = document.getElementById('btnConsultar');
      
      // Validação frontend
      if (!filial || !numeroNF) {
        resultadoDiv.innerHTML = `<div class="error"><p>Preencha todos os campos!</p></div>`;
        resultadoDiv.style.display = 'block';
        return;
      }
      
      // Mostra loading
      resultadoDiv.style.display = 'none';
      loadingDiv.style.display = 'block';
      btn.disabled = true;
      
      // Envia dados como OBJETO
      google.script.run
        .withSuccessHandler(function(res) {
          loadingDiv.style.display = 'none';
          btn.disabled = false;
          
          if (!res.success) {
            resultadoDiv.innerHTML = `<div class="error"><p>${res.message}</p></div>`;
          } else if (res.encontrada) {
            resultadoDiv.innerHTML = `
              <div class="success">
                <h3>NF Encontrada</h3>
                <p><strong>Número:</strong> ${res.dados.numeroNF}</p>
                <p><strong>Data:</strong> ${res.dados.dataRecebimento}</p>
                <p><strong>Status:</strong> ${res.dados.status}</p>
              </div>
            `;
          } else {
            resultadoDiv.innerHTML = `<div class="warning"><p>${res.message}</p></div>`;
          }
          
          resultadoDiv.style.display = 'block';
        })
        .withFailureHandler(function(err) {
          loadingDiv.style.display = 'none';
          btn.disabled = false;
          resultadoDiv.innerHTML = `<div class="error"><p>Erro: ${err.message}</p></div>`;
          resultadoDiv.style.display = 'block';
        })
        .consultarNF({  // Envia como objeto
          filial: filial,
          numeroNF: numeroNF
        });
    });
  </script>
</body>
</html>
