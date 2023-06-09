<template>
    <div class="Container">
        <div class="InformacoesContainer">
            <div class="Icone">
                <i :class="icone"></i>
            </div>
            <div class="DadosGerais">
                <div class="dados">
                    <div style="font-weight: bold;">
                        {{ titulo }}
                    </div>
                    <div>
                        Transportadora: {{ frete.empresa.transportadora }}
                    </div>
                    <div>
                        Tempo: {{ frete.empresa.tempoEstimado }}
                    </div>
                </div>
            </div>
        </div>
        <div class="PrecoContainer">
            <div>
                <div style="font-weight: bold;">
                    Preço
                </div>
                <div>
                    {{ frete.custoTotal }}
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'CardResult',
    props: {
        frete: {
            Object,
            required: true
        },
        titulo: String,
        icone: String,
    }
}

</script>

<style>
.Container {
    width: 100%;
    display: flex;
    justify-content: space-between;
    margin-bottom: inherit;
}

.InformacoesContainer {
    display: flex;
    background-color: rgb(228, 228, 228);
    width: 70%;
    border-radius: 5%;
}

.Icone {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 3em;
    background-color: #00aca6;
    padding: .5em;
    border-radius: 5%;
}

.DadosGerais {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100%;
}

.dados {
    width: inherit;
    margin-left: 3em;
}

.PrecoContainer {
    background-color: rgb(228, 228, 228);
    width: 25%;
    border-radius: 5%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}
</style>