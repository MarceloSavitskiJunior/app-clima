
<template>
    <div>
      <button class="btn btn-info" @click="obterLocalizacao">Usar minha localização</button>
    </div>
  </template>
  
  <script>
  export default {
    methods: {
      // metodo que vai lancar o pedido pro usuario compartilhar a loc dele e mandar
      // as coordenadas pro componente pai
      obterLocalizacao() {
        if ("geolocation" in navigator) {
          navigator.geolocation.getCurrentPosition(
            this.onLocalizacaoSucesso,
            this.onLocalizacaoErro
          );
        } else {
          alert("Sua geolocalização não é suportada pelo seu navegador.");
        }
      },
      onLocalizacaoSucesso(posicao) {
        const latitude = posicao.coords.latitude;
        const longitude = posicao.coords.longitude;
  
        this.$emit("localizacao-obtida", { latitude, longitude });
      },
      onLocalizacaoErro(erro) {
        console.error("Erro ao obter a localização:", erro.message);
      }
    }
  };
  </script>
  
  <style scoped>
  </style>
  