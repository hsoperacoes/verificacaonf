<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
      background-color: #f5f5f5;
    }
    .container {
      background: white;
      padding: 30px;
      border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    h1 {
      color: #2c3e50;
      text-align: center;
      margin-bottom: 30px;
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
      padding: 12px;
      border: 1px solid #ddd;
      border-radius: 6px;
      font-size: 16px;
      box-sizing: border-box;
    }
    button {
      background-color: #3498db;
      color: white;
      padding: 12px 20px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 16px;
      width: 100%;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #2980b9;
    }
    #resultado {
      margin-top: 30px;
      padding: 20px;
      border-radius: 6px;
      display: none;
    }
    .nf-encontrada {
      background-color: #e8f8f5;
      border-left: 5px solid #2ecc71;
    }
    .nf-nao-encontrada {
      background-color: #fdedec;
      border-left: 5px solid #e74c3c;
    }
    .status-recebida {
      color: #27ae60;
      font-weight: bold;
    }
    .status-pendente {
      color: #e67e22;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Consulta de Notas Fiscais</h1>
    
    <form id="consultaForm">
      <div class="form-group">
        <label for="filial">Filial:</label>
        <select id="filial" name="filial" required>
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
        <input type="text" id="numeroNF" name="numeroNF" required 
               placeholder="Digite o número completo da NF">
      </div>
      
      <button type="submit">Consultar</button>
    </form>
    
    <div id="resultado">
      <h2 id="resultado-titulo"></h2>
      <div id="detalhes-nf"></div>
    </div>
  </div>

  <script>
    document.getElementById('consultaForm').addEventListener('submit', function(e) {
      e.preventDefault();
      
      var filial = document.getElementById('filial').value;
      var numeroNF = document.getElementById('numeroNF').value;
      var btn = document.querySelector('button[type="submit"]');
      
      btn.disabled = true;
      btn.textContent = 'Consultando...';
      
      google.script.run.withSuccessHandler(function(result) {
        var resultadoDiv = document.getElementById('resultado');
        var titulo = document.getElementById('resultado-titulo');
        var detalhes = document.getElementById('detalhes-nf');
        
        resultadoDiv.style.display = 'block';
        
        if (result.encontrada) {
          resultadoDiv.className = 'nf-encontrada';
          titulo.textContent = '✓ Nota Fiscal Encontrada';
          
          var statusClass = result.dados.status === 'RECEBIDA' ? 
                         'status-recebida' : 'status-pendente';
          
          detalhes.innerHTML = `
            <p><strong>Filial:</strong> ${filial}</p>
            <p><strong>Número NF:</strong> ${numeroNF}</p>
            <p><strong>Status:</strong> <span class="${statusClass}">${result.dados.status}</span></p>
            ${result.dados.dataRecebimento ? 
              `<p><strong>Recebida em:</strong> ${result.dados.dataRecebimento}</p>` : 
              '<p><strong>Recebimento:</strong> Pendente</p>'}
          `;
        } else {
          resultadoDiv.className = 'nf-nao-encontrada';
          titulo.textContent = '✗ Nota Fiscal Não Encontrada';
          detalhes.innerHTML = `
            <p>Não encontramos a NF <strong>${numeroNF}</strong> na filial <strong>${filial}</strong>.</p>
            <p>Verifique o número ou consulte o responsável.</p>
          `;
        }
        
        btn.disabled = false;
        btn.textContent = 'Consultar';
        
      }).withFailureHandler(function(error) {
        alert('Erro: ' + error.message);
        btn.disabled = false;
        btn.textContent = 'Consultar';
      }).consultarNF(filial, numeroNF);
    });
  </script>
</body>
</html>
