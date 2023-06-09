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
            {{ appName }}
          </div>
        </div>

        <form @submit="onSubmit">
          <div class="Container">
            <div class="ContainerForm">
              <div class="itemForm">
                <i class="bi bi-globe-americas iconForm"></i>
                <span class="titleForm">Insira o destino e o peso</span>
              </div>
              <b-form-group label="Destino: " class="itemForm">
                <b-form-select v-model="selected" :options="cidades"></b-form-select>
              </b-form-group>

              <b-form-group label="Peso: " class="itemForm">
                <b-form-input id="input-1" v-model="peso" type="number" placeholder="Insira o peso em kg"></b-form-input>
              </b-form-group>

              <b-button type="submit" class="btnForm">Analisar</b-button>

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

            <div class="ContainerResult" style="font-size: 24px;" v-if="!moreCheapest && !moreFastest">
              Nenhum dado selecionado.
            </div>
            <div class="ContainerResult" v-if="moreCheapest && moreFastest">
              <div style="margin-right: auto; margin-bottom: inherit;">
                Estas são as melhores alternativas de frete que encontramos para você
              </div>
              <div class="ContainerMenorValor">
                <div class="dados">
                  <div class="imagem">
                    <i class="bi bi-cash-coin"></i>
                  </div>
                  <div class="informacoes">
                    <div class="divInformacoes">
                      <div style="font-weight: bold;">
                        Frete com menor valor
                      </div>
                      <div>
                        Transportadora: {{ moreCheapest.company.transportadora }}
                      </div>
                      <div>
                        Tempo: {{ moreCheapest.company.tempoEstimado }}
                      </div>
                    </div>
                  </div>
                </div>
                <div class="preco">
                  <div>
                    <div style="font-weight: bold;">
                      Preço
                    </div>
                    <div>
                      {{ moreCheapest.totalCost }}
                    </div>
                  </div>
                </div>
              </div>

              <div class="ContainerMenorValor">
                <div class="dados">
                  <div class="imagem">
                    <i class="bi bi-cash-coin"></i>
                  </div>
                  <div class="informacoes">
                    <div class="divInformacoes">
                      <div style="font-weight: bold;">
                        Frete mais rápido
                      </div>
                      <div>
                        Transportadora: {{ moreFastest.company.transportadora }}
                      </div>
                      <div>
                        Tempo: {{ moreFastest.company.tempoEstimado }}
                      </div>
                    </div>
                  </div>
                </div>
                <div class="preco">
                  <div>
                    <div style="font-weight: bold;">
                      Preço
                    </div>
                    <div>
                      {{ moreFastest.totalCost }}
                    </div>
                  </div>
                </div>
              </div>
              <b-button v-on:click="limpar" class="btnDelete">Limpar</b-button>
            </div>
          </div>
        </form>
      </b-card>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      appName: '',
      selected: null,
      cidades: [],
      peso: null,
      apiResult: null,
      moreCheapest: null,
      moreFastest: null,
      modalShow: false
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

    abrirModalCampoVazio() {
      this.modalShow = true;
    },

    onSubmit(event) {
      event.preventDefault()
      if (!this.selected || !this.peso) {
        this.abrirModalCampoVazio()
        return
      }
      this.moreCheapest = this.calcularFreteMaisBarato(this.selected, this.peso)
      this.moreFastest = this.calculateFastestDelivery(this.selected, this.peso)
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
      const company = filteredQuotes[0];

      // Calcular o custo total do frete
      const cost = weight <= 100 ? company.cost_transport_light : company.cost_transport_heavy;
      const totalCost = parseFloat(cost.replace("R$ ", "")) * weight;

      // Retornar o frete mais barato e seu custo total
      return {
        company: {
          transportadora: company.name,
          tempoEstimado: company.lead_time,
        },
        totalCost: `R$ ${totalCost.toFixed(2)}`,
      };
    },

    calculateFastestDelivery(city, weight) {
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
      const totalCost = parseFloat(cost.replace("R$ ", "")) * weight;

      // Retornar o frete mais barato e seu custo total
      return {
        company: {
          transportadora: fastestQuote.name,
          tempoEstimado: fastestQuote.lead_time,
        },
        totalCost: `R$ ${totalCost.toFixed(2)}`,
      };
    },

    limpar(event) {
      event.preventDefault()
      this.selected = null
      this.peso = null
      this.moreCheapest = null
      this.moreFastest = null
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

.cardTitle {
  background-color: #00aca6;
  height: 4em;
  display: flex;
  align-items: center;
  padding: 1em;
}

.card-body {
  padding: 0;
}

.ContainerForm {
  background-color: rgb(237, 237, 237);
  width: 30%;
  height: 40rem;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 2em;
  flex-direction: column;
  border-radius: 5%;
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

.Container {
  display: flex;
}

.ContainerMenorValor {
  width: 100%;
  display: flex;
  justify-content: space-between;
  margin-bottom: inherit;
}

.imagem {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 3em;
  background-color: #00aca6;
  padding: .5em;
  border-radius: 5%;
}

.dados {
  display: flex;
  background-color: rgb(228, 228, 228);
  width: 70%;
  border-radius: 5%;
}

.iconTitleCard {
  font-size: 2.5rem;
  margin-right: 0.5rem;
}

.nameTitleCard {
  font-size: 1.5rem;
  font-weight: bold;
}

.informacoes {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.divInformacoes{
  width:inherit;
  margin-left: 3em;
}

.preco {
  background-color: rgb(228, 228, 228);
  width: 25%;
  border-radius: 5%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.iconForm {
  font-size: 2rem;
  margin-right: 0.5rem;
}

.MainContainer {
  margin: 5em auto;
}

.btnForm {
  background-color: #00aca6;
  border-color: #00aca6;
  width: 50%;
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

.btnForm:hover,
.btnForm:focus,
.btnForm:active,
.btnForm.active,
.modalBtn:hover,
.modalBtn:focus,
.modalBtn:active,
.modalBtn.active,
.open .dropdown-toggle.btnForm,
.open .dropdown-toggle.modalBtn {
  background-color: #038883;
  border-color: #038883;
}

.titleForm {
  font-weight: bold;
  font-size: x-large;
}

.itemForm {
  margin-bottom: 2em;
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
