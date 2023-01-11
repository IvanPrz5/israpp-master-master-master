<template>
  <v-container>
    <v-card-title class="text-h6 font-weight-light">
      Calculo por excel
    </v-card-title>
    <v-combobox v-model="añoData" :items="años" label="Año" outlined dense></v-combobox>
    <v-combobox v-model="periodoData" :items="periodo" label="Periodo" outlined dense></v-combobox>
    <v-combobox v-model="tipoCal" :items="itemsCal" label="Elije Calculadora" outlined dense></v-combobox>
    <v-file-input v-if="tipoCal.length > 0" id="archivo" label="File input" @change="subirExcel()" outlined
      dense></v-file-input>
    <v-data-table :items="itemsTable" :headers="headers" :items-per-page="5" class="elevation-1"></v-data-table>
    <div class="btn-container">
      <v-btn v-if="itemsTable.length > 0" id="btn-descargar" depressed color="primary" @click="descargaExcel()">
        Descargar Excel
      </v-btn>
    </div>
  </v-container>
</template>

<script>
import axios from "axios";
import readXlsFile from "read-excel-file";
import exportFromJSON from "export-from-json";

export default {
  name: "FormIsrExcel",
  data() {
    return {
      vacia: "",
      añoData: "",
      periodoData: "",
      isrData: "",
      netoData: "",
      dataApi: [],
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
      periodo: ["Mensual", "Quincenal", "Semanal"],
      itemsCal: ["Bruto a Neto", "Neto a Bruto"],
      tipoCal: "",
      //Items Api
      itemsFile: [],
      //Items Tabla
      itemsTable: [],
      headers: [],
    };
  },
  methods: {
    calculaBrutoExcel(ingresoData, tablasIsr) {
      for (let i = 0; i < tablasIsr.data.length; i++) {
        //console.log(tablasIsr.data[i].limInf);
        let limInf = tablasIsr.data[i].limInf;
        let porcentaje = tablasIsr.data[i].porcentaje;
        let cuota = tablasIsr.data[i].cuota;
        if (
          tablasIsr.data[i].limInf < ingresoData &&
          tablasIsr.data[i].limSup > ingresoData
        ) {
          let base = ingresoData - limInf;
          let impuestoMar = base * porcentaje;
          let isrData = impuestoMar + cuota;
          let neto = ingresoData - isrData;
          this.itemsTable.push({
            Bruto: ingresoData,
            Isr: isrData.toFixed(2),
            Neto: neto.toFixed(2),
          });
        }
      }
    },
    calculaNetoExcel(ingresoData, tablasIsr) {
      for (let i = 0; i < tablasIsr.data.length; i++) {
        let netoFloat = parseFloat(ingresoData);
        let netoFloatProducto = ingresoData * 2;
        let limInf = tablasIsr.data[i].limInf;
        let porcentaje = tablasIsr.data[i].porcentaje;
        let cuota = tablasIsr.data[i].cuota;
        if (
          tablasIsr.data[i].limInf < netoFloatProducto &&
          tablasIsr.data[i].limSup > netoFloatProducto
        ) {
          let base = netoFloatProducto - limInf;
          let impuestoMarginal = base * porcentaje;
          let isrData = impuestoMarginal + cuota;
          let netoFloatTemporal = netoFloat + isrData;
          //console.log(netoFloatTemporal);
          for (let j = 0; j < tablasIsr.data.length; j++) {
            for (let k = netoFloat; k < netoFloatTemporal; k++) {
              if (netoFloat < netoFloatTemporal) {
                netoFloatProducto = netoFloatTemporal;
                if (
                  tablasIsr.data[j].limInf < netoFloatProducto &&
                  tablasIsr.data[j].limSup > netoFloatProducto
                ) {
                  let base = netoFloatProducto - tablasIsr.data[j].limInf;
                  let impuestoMarginal = base * tablasIsr.data[j].porcentaje;
                  let isrData = impuestoMarginal + tablasIsr.data[j].cuota;
                  netoFloatTemporal = netoFloat + isrData;
                  if (netoFloatTemporal == netoFloatProducto) {
                    this.itemsTable.push({
                      Neto: ingresoData,
                      Isr: isrData.toFixed(2),
                      Bruto: netoFloatTemporal.toFixed(2),
                    });
                    break;
                  }
                }
              }
            }
          }
        }
      }
    },
    getTablasIsr(ingresoData) {
      axios
        .get(
          "http://localhost:8081/Isr/año/" +
          this.añoData +
          "/" +
          this.periodoData
        )
        .then((response) => {
          this.dataApi = response.data.data;
          if (this.tipoCal === "Bruto a Neto") {
            this.calculaBrutoExcel(ingresoData, response);
          } else if (this.tipoCal === "Neto a Bruto") {
            this.calculaNetoExcel(ingresoData, response);
          }
        });
    },
    subirExcel() {
      let inputFile = document.getElementById("archivo");
      readXlsFile(inputFile.files[0]).then((rows) => {
        this.itemsFile = rows;
        for (let i = 0; i < this.itemsFile.length; i++) {
          this.getTablasIsr(this.itemsFile[i].toString());
        }
        if (this.tipoCal === "Bruto a Neto") {
          //console.log(this.itemsTable);
          this.itemsTable.length = "";
          this.headers.length = "";
          this.headers.push(
            {
              text: "Ingreso Bruto",
              value: "Bruto",
            },
            {
              text: "Isr",
              value: "Isr",
            },
            {
              text: "Neto",
              value: "Neto",
            }
          );
          this.itemsTable.length = "";
        } else if (this.tipoCal === "Neto a Bruto") {
          this.itemsTable.length = "";
          this.headers.length = "";
          this.headers.push(
            {
              text: "Ingreso Neto",
              value: "Neto",
            },
            {
              text: "Isr",
              value: "Isr",
            },
            {
              text: "Bruto",
              value: "Bruto",
            }
          );
        }
      });
    },
    descargaExcel() {
      let data = this.itemsTable;
      let fileName = "libro";
      let exportType = exportFromJSON.types.csv;
      exportFromJSON({ data, fileName, exportType });
    },
  },
};
</script>

<style>
.btn-container {
  margin-top: 27px;
}
</style>
