<template>
    <div class="icone-clima">
      <img :src="iconeURL" class="card-image-top img-fluid" style="max-width: 100%;">
    </div>
</template>
  
<script>
  export default {
    props: {
      icone: String
    },
    computed: {
      iconeURL() {
        // faz um get http do icon usando o "id" passado anteriormente
        return `https://openweathermap.org/img/wn/${this.icone}.png`;
      }
    }
  };
</script>

<style scoped>
@import '../assets/styles.css'
</style>