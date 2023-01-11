<template>
  <v-container>
    <v-card-title class="text-h6 font-weight-light">
      Neto a Bruto
    </v-card-title>
    <v-combobox v-model="añoData" :items="años" label="Año" outlined dense></v-combobox>
    <v-combobox v-model="periodoData" :items="periodo" label="Periodo" outlined dense></v-combobox>
    <v-text-field v-model="netoData" label="Neto" aria-required="true" outlined dense></v-text-field>
    <div class="btn-container">
      <v-btn depressed color="primary" @click="calculaBruto()">
        Calcular
      </v-btn>
    </div>
    <v-text-field v-model="brutoData" label="Bruto" outlined dense readonly></v-text-field>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  name: "FormNetoBruto",
  data() {
    return {
      isrData: "",
      añoData: "",
      periodoData: "",
      netoData: "",
      brutoData: "",
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
      periodo: ["Mensual", "Quincenal", "Semanal", "Diario"],
    };
  },
  methods: {
    calculaBruto() {
      axios
        .get(
          "http://localhost:8081/Isr/año/" +
          this.añoData +
          "/" +
          this.periodoData
        )
        .then((response) => {
          this.result = response.data;
          for (let i = 0; i < this.result.length; i++) {
            let netoFloat = parseFloat(this.netoData);
            let netoFloatProducto = this.netoData * 2;
            let limInf = response.data[i].limInf;
            let porcentaje = response.data[i].porcentaje;
            let cuota = response.data[i].cuota;
            //console.log(response.data[i].limInf + "  -------  " + netoFloatProducto + "  ,   " + response.data[i].limSup );
            if (
              response.data[i].limInf < netoFloatProducto &&
              response.data[i].limSup > netoFloatProducto
            ) {
              let base = netoFloatProducto - limInf;
              let impuestoMarginal = base * porcentaje;
              let isrData = impuestoMarginal + cuota;
              let netoFloatTemporal = netoFloat + isrData;
              //console.log(netoFloatTemporal);
              for (let j = 0; j < this.result.length; j++) {
                for (let k = netoFloat; k < netoFloatTemporal; k++) {
                  if (netoFloat < netoFloatTemporal) {
                    netoFloatProducto = netoFloatTemporal;
                    if (
                      response.data[j].limInf < netoFloatProducto &&
                      response.data[j].limSup > netoFloatProducto
                    ) {
                      /* console.log(response.data[j].limInf);
                      console.log(response.data[j].limSup); */
                      let base = netoFloatProducto - response.data[j].limInf;
                      let impuestoMarginal = base * response.data[j].porcentaje;
                      let isrData = impuestoMarginal + response.data[j].cuota;
                      netoFloatTemporal = netoFloat + isrData;
                      if (netoFloatTemporal == netoFloatProducto) {
                        //console.log(netoFloatTemporal.toFixed(2) + "**");
                        this.brutoData = netoFloatTemporal;
                        this.brutoData = this.brutoData.toFixed(2);
                        break;
                      }
                    }
                  }
                }
              }
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
}
</style>
