<template>
    <div class="ContainerForm">
        <form @submit="emitSubmit">
            <div class="itemForm">
                <i class="bi bi-globe-americas iconForm"></i>
                <span class="titleForm">Insira o destino e o peso</span>
            </div>
            <b-form-group label="Destino: " class="itemForm">
                <b-form-select v-model="cidadeSelecionada" :options="cidades" @change="emitCidade"></b-form-select>
            </b-form-group>

            <b-form-group label="Peso: " class="itemForm">
                <b-form-input v-model="pesoFrete" type="number" placeholder="Insira o peso em kg"
                    @change="emitPeso"></b-form-input>
            </b-form-group>

            <b-button type="submit" class="btnForm">Analisar</b-button>
        </form>
    </div>
</template>

<script>
export default {
    name: 'FormFrete',
    props: {
        cidades: {
            type: Array,
        },
        clear: Boolean,
    },
    watch: {
        clear() {
            this.cidadeSelecionada = null;
            this.pesoFrete = null;
        }
    },
    data() {
        return {
            cidadeSelecionada: null,
            pesoFrete: null,
        };
    },
    methods: {
        emitCidade() {
            this.$emit('cidade-selecionada', this.cidadeSelecionada);
        },
        emitPeso() {
            this.$emit('peso-selecionado', this.pesoFrete);
        },
        emitSubmit(event) {
            event.preventDefault();
            this.$emit('submit');
        }
    }
}

</script>

<style>
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

.iconForm {
    font-size: 2rem;
    margin-right: 0.5rem;
}

.titleForm {
    font-weight: bold;
    font-size: x-large;
}

.btnForm {
    background-color: #00aca6 !important;
    border-color: #00aca6 !important;
    width: 50% !important;
}


.btnForm:hover,
.btnForm:focus,
.btnForm:active,
.btnForm.active,
.modalBtn:hover {
    background-color: #038883 !important;
    border-color: #038883 !important;
}

.itemForm {
    margin-bottom: 2em;
}
</style>