<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>

    <div>
      <b-card class="MainContainer" style="max-width: 80rem; border-radius: 5%;">
        <div class="cardTitle">
          <i class="bi bi-truck iconTitleCard"></i>
          <div class="nameTitleCard">
            Melhor Frete
          </div>
        </div>

          <div class="flex">
            <FormFrete @cidade-selecionada="handleCidadeSelecionada" @peso-selecionado="handlePesoFrete"
              :cidades="cidades" @submit="onSubmit" :delete="limpar" :clear="clear"/>

            <div class="ContainerResult ContainerResultEmpty" v-if="!empresaBarata && !empresaRapida">
              Nenhum dado selecionado.
            </div>
            <div class="ContainerResult" v-if="empresaBarata && empresaRapida">
              <div style="margin-right: auto; margin-bottom: inherit;">
                Estas são as melhores alternativas de frete que encontramos para você
              </div>

              <CardResult :frete="empresaBarata" titulo="Frete com menor valor" icone="bi bi-cash-coin" />
              <CardResult :frete="empresaRapida" titulo="Frete mais rápido" icone="bi bi-clock" />

              <b-button v-on:click="limpar" class="btnDelete">Limpar</b-button>
            </div>
          </div>
      </b-card>
    </div>
    <b-modal v-model="modalShow" :hide-footer="true" :hide-header="true">
      <div class="modalIcon">
        <i class="bi bi-exclamation-octagon"></i>
      </div>
      <div class="modalText">
        Insira os valores para realizar a análise.
      </div>
      <b-button class="modalBtn" @click="modalShow = false">Fechar</b-button>
    </b-modal>
  </div>
</template>

<script>
import axios from 'axios'

import CardResult from './CardResult.vue'
import FormFrete from './FormFrete.vue'

export default {
  name: 'BestTransport',
  components: {
    CardResult,
    FormFrete,
  },
  data() {
    return {
      appName: '',
      selected: null,
      cidades: [],
      peso: null,
      apiResult: null,
      empresaBarata: null,
      empresaRapida: null,
      modalShow: false,
      clear: false,
    }
  },
  async created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = 'Melhor Frete'

    let result = await axios.get("http://localhost:3000/transport")
    this.apiResult = result.data
    let cidades = result.data.map(result => result.city);

    cidades.unshift({ value: null, text: "Selecione a cidade de destino" });
    this.cidades = [...new Set(cidades)];
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
      console.log(this.appName)
    },

    handleCidadeSelecionada(value) {
      this.selected = value
    },

    handlePesoFrete(value) {
      this.peso = value
    },

    onSubmit() {
      if (!this.selected || !this.peso) {
        this.modalShow = true;
        return
      }
      this.empresaBarata = this.calcularFreteMaisBarato(this.selected, this.peso)
      this.empresaRapida = this.calcularFreteMaisRapido(this.selected, this.peso)
    },

    calcularFreteMaisBarato(city, weight) {
      // Filtrar as cotações com base na cidade fornecida
      const filteredQuotes = this.apiResult.filter((quote) => quote.city === city);

      // Ordenar as cotações com base no custo apropriado para o peso fornecido
      filteredQuotes.sort((a, b) => {
        const costA = weight <= 100 ? parseFloat(a.cost_transport_light.replace("R$ ", "")) : parseFloat(a.cost_transport_heavy.replace("R$ ", ""));
        const costB = weight <= 100 ? parseFloat(b.cost_transport_light.replace("R$ ", "")) : parseFloat(b.cost_transport_heavy.replace("R$ ", ""));
        return costA - costB;
      });

      // Obter a cotação mais barata
      const empresa = filteredQuotes[0];

      // Calcular o custo total do frete
      const cost = weight <= 100 ? empresa.cost_transport_light : empresa.cost_transport_heavy;
      const custoTotal = parseFloat(cost.replace("R$ ", "")) * weight;

      // Retornar o frete mais barato e seu custo total
      return {
        empresa: {
          transportadora: empresa.name,
          tempoEstimado: empresa.lead_time,
        },
        custoTotal: `R$ ${custoTotal.toFixed(2)}`,
      };
    },

    calcularFreteMaisRapido(city, weight) {
      // Filtrar as cotações com base na cidade fornecida
      const filteredQuotes = this.apiResult.filter((quote) => quote.city === city);

      // Ordenar as cotações pelo tempo de entrega
      filteredQuotes.sort((a, b) => {
        const timeA = parseInt(a.lead_time);
        const timeB = parseInt(b.lead_time);
        return timeA - timeB;
      });

      // Obter a cotação com o tempo de entrega mais rápido
      const fastestQuote = filteredQuotes[0];

      // Calcular o custo total do frete
      const cost = weight <= 100 ? fastestQuote.cost_transport_light : fastestQuote.cost_transport_heavy;
      const custoTotal = parseFloat(cost.replace("R$ ", "")) * weight;

      // Retornar o frete mais barato e seu custo total
      return {
        empresa: {
          transportadora: fastestQuote.name,
          tempoEstimado: fastestQuote.lead_time,
        },
        custoTotal: `R$ ${custoTotal.toFixed(2)}`,
      };
    },

    limpar(event) {
      event.preventDefault()
      this.selected = null
      this.peso = null
      this.empresaBarata = null
      this.empresaRapida = null
      this.clear = true
    },
  },
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
.card-body {
  padding: 0;
}

.cardTitle {
  background-color: #00aca6;
  height: 4em;
  display: flex;
  align-items: center;
  padding: 1em;
}


.ContainerResult {
  width: 60%;
  height: 40rem;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 2em;
  flex-direction: column;
}

.ContainerResultEmpty {
  font-size: 24px;
}

.flex {
  display: flex;
}

.iconTitleCard {
  font-size: 2.5rem;
  margin-right: 0.5rem;
}

.nameTitleCard {
  font-size: 1.5rem;
  font-weight: bold;
}

.MainContainer {
  margin: 5em auto;
}

.btnDelete {
  background-color: #e74c3c;
  border-color: #e74c3c;
  width: 50%;
  margin-top: auto;
  margin-left: auto;
}

.btnDelete:hover,
.btnDelete:focus,
.btnDelete:active,
.btnDelete.active,
.open .dropdown-toggle.btnDelete {
  background-color: #c64031;
  border-color: #c64031;
}

.modalBtn:hover,
.modalBtn:focus,
.modalBtn:active,
.modalBtn.active {
  background-color: #038883;
  border-color: #038883;
}

.modalIcon {
  font-size: 3rem;
  font-weight: bold;
}

.modalIcon i {
  margin-left: 45%;
}

.modalBtn {
  background-color: #00aca6;
  border-color: #00aca6;
  margin-left: 41%;
  margin-top: 1em
}

.modalText {
  font-size: 1.5rem;
  font-weight: bold;
}
</style>
