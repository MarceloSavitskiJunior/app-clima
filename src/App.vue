<template>
  <div id="main" :class="isDia ? 'day' : 'night'">
    <div class="container my-5" style="max-width: 400px; min-width: 360px">
      <h1 class="titulo text-center">Clima's</h1>
      <form class="search-location" v-on:submit.prevent="getClima" v-on:focusout.prevent="getClima">
        <input
          type="text"
          class="form-control text-muted br-2 p-4 shadow-sm"
          placeholder="Digite a cidade..."
          v-model="cidadePesquisada"
          autocomplete="off"
        />
      </form>
      <p class="text-center my-3" v-if="!isCidadeEncontrada">Não encontramos uma cidade com esse nome :(</p>
      <div
        class="card rounded my-3 shadow-lg fundo-card overflow-hidden" :class="getFundoCard()"
        v-if="isVisible">
        <div>
          <div style="display: flex; justify-content: center;">
            <IconeClima :icone="icon" />
          </div>
        </div>

        <div class="card-top text-center" style="margin-bottom: 8rem">
          <div class="nome-cidade my-3">
            <p>{{ clima.nomeCidade }}</p>
            <p class="">{{ clima.pais }}</p>
          </div>
        </div>

        <div class="card-body">
          <div class="card-mid">
            <div class="row">
              <div class="col-12 text-center temp">
                <span>{{ clima.temperatura }}&deg;C</span>
                <p class="my-4">{{ clima.descricao }}</p>
              </div>
            </div>
          </div>

          <div class="card-bottom px-5 py-4 row">
            <div class="col text-center">
              <p>{{ clima.velocidadeVento }}</p>
              <span>Velocidade do Vento (km/h)</span>
            </div>
            <div class="col text-center">
              <p>{{ clima.umidade }}%</p>
              <span>Umidade</span>
            </div>
          </div>

        </div>
      </div>
        <div class="col text-center mt-2">
          <LocalizacaoComponent @localizacao-obtida="processarLocalizacao" />
        </div>
    </div>
  </div>
</template>

<script>
import LocalizacaoComponent from './componentes/LocalizarUsuarioComponent.vue'
import IconeClima from './componentes/IconeClima.vue'

export default {
  data() {
    return {
      isCidadeEncontrada: true,
      isVisible: false,
      isTrovejando: false,
      isNublado: false,
      isEnsolarado: false,
      isNoiteAberta: false,
      isNevando: false,
      isDia: true,
      cidadePesquisada: "",
      icon: "",
      clima: {
        nomeCidade: "",
        pais: "",
        temperatura: 0,
        descricao: "",
        velocidadeVento: "",
        umidade: "",
      },
    };
  },
  components: {
    LocalizacaoComponent,
    IconeClima
  },
  methods: {
    // aqui eu criei esse metodo para caso o usuario queira usar sua propria localizacao
    // ele serve para buscar os mesmos dados que são buscados mandando uma cidade, porém enviando uma coordenada
    async processarLocalizacao(coordenadas) {
      const key = "b9e3dc34e2928ddf5a40550aa4f91364";
      const baseURL = `https://api.openweathermap.org/data/2.5/weather?lat=${coordenadas.latitude}&lon=${coordenadas.longitude}&appid=${key}`;
      
      try {
        const response = await fetch(baseURL);

        if (!response.ok) {
          throw new Error(`Erro de HTTP! Status: ${response.status}`);
        }

        const responseData = await response.json();
        // Chama o método getClima passando os dadosLocais obtidos pelas coordenadas
        const local = this.getClima(responseData);
        return local;
        
      } catch (error) {
        console.error("Erro ao processar localização:", error);
        throw error;
      }
    },
    // Método para obter dados de clima por cidade
    async obterDadosClimaPorCidade(cidade) {
      // chave pra usar a api (dura 15 dias a partir do dia 15 de janeiro de 2024)
      const key = "b9e3dc34e2928ddf5a40550aa4f91364";
      // request
      const baseURL = `https://api.openweathermap.org/data/2.5/weather?q=${cidade}&appid=${key}&units=metric`;

      // aqui mando a url e ja atribuo na variavel da resposta
      const response = await fetch(baseURL);

      if (!response.ok) {
        throw new Error(`Erro de HTTP! Status: ${response.status}`);
      }

      // retorno um json
      return await response.json();
    },
    async getClima(dadosLocais) {
      try {
        let data = "";
        if (dadosLocais.name) {
          // busca pelos dados locais (dadosLocais.name é o nome da cidade)
          data = await this.obterDadosClimaPorCidade(dadosLocais.name);
        } else {
          // busca normalmente
          data = await this.obterDadosClimaPorCidade(this.cidadePesquisada);
        }

        // set dos valores que utilizo para mostrar na interface
        this.clima.nomeCidade = data.name;
        this.clima.pais = data.sys.country;
        this.clima.temperatura = Math.round(data.main.temp);
        this.clima.descricao = this.getDescricao(data.weather[0].main);
        this.clima.velocidadeVento = data.wind.speed;
        this.clima.umidade = Math.round(data.main.humidity);

        // set dos valores booleanos pra usar depois em metodos 
        // de retorno de string de descriçoes e etc
        const climaAtual = data.weather[0].main;
        this.atualizarClima(climaAtual);

        // preciso da hora atual para descobrir se é noite ou dia no local
        const horaAtualUnix = Math.round(new Date().getTime() / 1000);

        const sunsetUnix = data.sys.sunset;
        const sunriseUnix = data.sys.sunrise;

        // aqui decide se é dia ou noite baseado no por do sol e no nascer do sol do local pesquisado
        // se a hora da regiao é maior q o nascer da outra e menor que o por do sol, eh dia
        this.isDia = horaAtualUnix > sunriseUnix && horaAtualUnix < sunsetUnix;

        // recebe o "id" do icon pra pesquisar no componente do icon
        this.icon = data.weather[0].icon;

        this.isVisible = true;
        this.isCidadeEncontrada = true;
      } catch (error) {
        console.log(error);
        this.isCidadeEncontrada = false;
        this.isVisible = false;
      }
    },
    // atualiza as variaveis de clima
    atualizarClima(climaAtual) {
      this.isDia = !climaAtual.includes("n")
      this.isTrovejando = climaAtual.includes("Thunderstorm") || climaAtual.includes("Rain");
      this.isNublado = climaAtual.includes("Clouds");
      this.isEnsolarado = climaAtual.includes("Clear");
      this.isNoiteAberta = !climaAtual.includes("Clouds");
      this.isNevando = climaAtual.includes("Snow");
    },
    // get das descricoes que vao aparecer
    getDescricao(descricao) {
      if (descricao.includes('Snow')) return 'Nevando';
      if (descricao.includes('Clouds')) return 'Nublado';
      if (descricao.includes('Rain')) return 'Chovendo';
      if (descricao.includes('Clear')) return 'Céu Limpo';
      if (descricao.includes('Thunderstorm')) return 'Trovoadas';
      return 'Clima não detectado';
    },
    // get do clima que vai aparecer
    getTipoClima() {
      if (this.isTrovejando) return "Tempestade";
      if (this.isNublado) return "Nublado";
      if (this.isEnsolarado) return "Ensolarado";
      if (this.isNoiteAberta) return "Noite Aberta";
      if (this.isNevando) return "Nevando";
      return "Desconhecido";
    },
    // define a classe css que vai mudar a cor do card
    getFundoCard() {
      if (this.isTrovejando) return "chovendo";
      if (this.isNublado) return "nublado";
      if (this.isEnsolarado) return "ensolarado";
      if (this.isNoiteAberta) return "noite";
      if (this.isNevando) return "nevando";
      return 'fundo-card';
    },
  },
};
</script>

<style scoped>
@import './assets/styles.css'
</style>