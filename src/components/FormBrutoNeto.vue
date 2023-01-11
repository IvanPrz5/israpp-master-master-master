<template>
  <v-container class="container">
    <v-card-title>
      <span class="text-h6 font-weight-light">Bruto a Neto</span>
    </v-card-title>
    <div class="netoForm" cols="12">
      <v-combobox v-model="añoData" :items="años" label="Año" outlined dense></v-combobox>
      <v-combobox v-model="periodoData" :items="periodo" label="Periodo" outlined dense></v-combobox>
      <v-text-field v-if="periodoData == 'Diario'" v-model="diasData" label="Días a calcular" outlined
        dense></v-text-field>
      <v-text-field v-model="ingresoBruto" label="Bruto" aria-required="true" outlined dense></v-text-field>
      <div class="btn-container">
        <v-btn depressed color="primary" @click="calculaNeto()">
          Calcular
        </v-btn>
      </div>
      <v-text-field v-model="isrData" label="ISR" outlined dense readonly></v-text-field>
      <v-text-field v-model="netoData" label="Neto" outlined dense readonly></v-text-field>
    </div>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  name: "FormBrutoNeto",
  data() {
    return {
      vista: "",
      añoData: "",
      periodoData: "",
      diasData: "",
      ingresoBruto: "",
      isrData: "",
      netoData: "",
      //Items
      años: [
        "2015",
        "2016",
        "2017",
        "2018",
        "2019",
        "2020",
        "2021",
        "2022",
        "2023",
      ],
      periodo: ["Anual", "Mensual", "Quincenal", "Semanal", "Diario"],
      dataApi: [],
    };
  },
  methods: {
    calculaNeto() {
      axios
        .get(
          "http://localhost:8081/Isr/año/" +
          this.añoData +
          "/" +
          this.periodoData
        )
        .then((response) => {
          this.dataApi = response.data;
          for (let i = 0; i < this.dataApi.length; i++) {
            let limInf = this.dataApi[i].limInf;
            let porcentaje = this.dataApi[i].porcentaje;
            let cuota = this.dataApi[i].cuota;
            if (
              this.dataApi[i].limInf < this.ingresoBruto &&
              this.dataApi[i].limSup > this.ingresoBruto
            ) {
              let base = this.ingresoBruto - limInf;
              let impuestoMar = base * porcentaje;
              this.isrData = impuestoMar + cuota;
              this.isrData = this.isrData.toFixed(2);
              this.netoData = this.ingresoBruto - this.isrData;
              //this.netoData = parseFloat(this.ingresoBruto) + parseFloat(this.isrData);
              this.netoData = this.netoData.toFixed(2);
            }
          }
        });
    },
  },
};
</script>

<style>
.btn-container {
  margin-bottom: 27px;
  /* color: #48ad6a */
}
</style>
