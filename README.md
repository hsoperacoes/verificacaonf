<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    /* Mantenha os estilos anteriores ou use esses simplificados */
    body { font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; }
    .form-group { margin-bottom: 15px; }
    select, input { width: 100%; padding: 10px; margin-top: 5px; }
    button { background: #4285f4; color: white; border: none; padding: 12px; width: 100%; cursor: pointer; }
    #resultado { margin-top: 20px; padding: 15px; border-radius: 4px; display: none; }
    .ok { background: #e6f4ea; border-left: 4px solid #34a853; }
    .pendente { background: #fce8e6; border-left: 4px solid #ea4335; }
    .nao-encontrado { background: #f8f9fa; border-left: 4px solid #9aa0a6; }
  </style>
</head>
<body>
  <h1>Consulta de Notas Fiscais</h1>
  <form id="consultaForm">
    <div class="form-group">
      <label>Filial:</label>
      <select id="filial" required>
        <option value="">Selecione</option>
        <option value="ARTUR">ARTUR (Colunas A-D)</option>
        <option value="FLORIANO">FLORIANO (Colunas E-H)</option>
        <option value="JOTA">JOTA (Colunas I-L)</option>
        <option value="MODA">MODA (Colunas M-P)</option>
        <option value="PONTO">PONTO (Colunas Q-T)</option>
      </select>
    </div>
    
    <div class="form-group">
      <label>Número da NF:</label>
      <input type="text" id="numeroNF" required>
    </div>
    
    <button type="submit">Consultar</button>
  </form>
  
  <div id="resultado"></div>

  <script>
    document.getElementById('consultaForm').addEventListener('submit', function(e) {
      e.preventDefault();
      var filial = document.getElementById('filial').value;
      var nf = document.getElementById('numeroNF').value;
      var btn = this.querySelector('button');
      
      btn.disabled = true;
      btn.textContent = 'Buscando...';
      
      google.script.run.withSuccessHandler(function(res) {
        var resultado = document.getElementById('resultado');
        resultado.style.display = 'block';
        
        if (res.encontrada) {
          resultado.className = res.dados.status === 'RECEBIDA' ? 'ok' : 'pendente';
          resultado.innerHTML = `
            <h3>NF ${nf} - ${filial}</h3>
            <p>Status: <strong>${res.dados.status}</strong></p>
            ${res.dados.dataRecebimento ? 
              `<p>Recebida em: ${res.dados.dataRecebimento}</p>` : 
              '<p>Data recebimento: Pendente</p>'}
          `;
        } else {
          resultado.className = 'nao-encontrado';
          resultado.innerHTML = `
            <h3>NF não encontrada</h3>
            <p>A nota fiscal ${nf} não consta no sistema para a filial ${filial}</p>
          `;
        }
        
        btn.disabled = false;
        btn.textContent = 'Consultar';
        
      }).withFailureHandler(function(err) {
        alert('Erro: ' + err.message);
        btn.disabled = false;
        btn.textContent = 'Consultar';
      }).consultarNF(filial, nf);
    });
  </script>
</body>
</html>
