<script>
  // Adicione este estilo para o loading
  const style = document.createElement('style');
  style.innerHTML = `
    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
    .loading-spinner {
      border: 4px solid #f3f3f3;
      border-top: 4px solid #3498db;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      animation: spin 1s linear infinite;
      margin: 20px auto;
    }
    .progress-text {
      text-align: center;
      margin-top: 10px;
      color: #555;
    }
  `;
  document.head.appendChild(style);

  // Modifique a função de consulta
  form.addEventListener('submit', function(e) {
    e.preventDefault();
    
    const resultadoDiv = document.getElementById('resultado');
    resultadoDiv.innerHTML = `
      <div class="loading-spinner"></div>
      <div class="progress-text">Consultando banco de dados...</div>
    `;
    resultadoDiv.style.display = 'block';
    
    // Restante do código...
  });
</script>
