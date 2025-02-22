<template>
    <div class="iframe-container">
      <iframe
        src="https://whatsapp-nu-one.vercel.app"
        frameborder="0"
        class="iframe-content"
        title="Chat Manager"
      ></iframe>
    </div>
  </template>
  
  <script>
  export default {
    name: 'IframeComponent',
  };
  </script>
  
  <style scoped>
  html, body {
    margin: 0;
    padding: 0;
    height: 100%; /* Garante que o body ocupe 100% da altura */
  }
  
  .iframe-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100vh; /* Ocupar a altura total da tela */
    overflow: hidden;
  }
  
  .iframe-content {
    width: 100%;
    height: 100%;
    border: none; /* Remove bordas do iframe */
  }
  </style>
  