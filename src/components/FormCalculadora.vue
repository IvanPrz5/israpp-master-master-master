<template>
  <v-container class="containerCal">
    <div class="calForm">
      <v-combobox v-model="añoData" :items="años" label="Año" outlined dense></v-combobox>
      <v-combobox v-model="periodoData" :items="periodo" label="Periodo" outlined dense></v-combobox>
      <v-text-field v-if="periodoData == 'Diario'" label="Días a calcular" outlined dense></v-text-field>
      <v-combobox v-model="tipoCal" :items="itemsCal" label="Tipo Calculadora" outlined dense></v-combobox>
      <v-text-field v-model="ingresoData" v-if="tipoCal == 'Bruto a Neto' || tipoCal == 'Neto a Bruto'" label="Bruto"
        aria-required="true" outlined dense></v-text-field>
      <div v-if="tipoCal == 'Bruto a Neto' || tipoCal == 'Neto a Bruto'" class="btn-container">
        <v-btn depressed color="primary" @click="calculaIngreso()">
          Calcular
        </v-btn>
      </div>
      <div v-if="tipoCal == 'Bruto a Neto' || tipoCal == 'Neto a Bruto'">
        <v-text-field v-model="isrData" :label="ISR" outlined dense readonly></v-text-field>
        <v-text-field v-model="resultData" label="Neto" outlined dense readonly></v-text-field>
      </div>
    </div>
    <div>
      <v-combobox v-if="tipoCal == 'Excel'" v-model="tipoCalExcel" :items="itemsCalExcel" label="Tipo Calculadora"
        outlined dense></v-combobox>
      <v-file-input v-if="tipoCalExcel.length > 0" id="archivo" label="File input" @change="subirExcel()" outlined
        dense></v-file-input>
      <v-data-table v-if="tipoCalExcel.length > 0" :items="itemsTable" :headers="headers" :items-per-page="5"
        class="elevation-1"></v-data-table>
    </div>
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
  name: "FormCalculadora",
  components: {},
  data() {
    return {
      ISR:"Hola",
      añoData: "",
      periodoData: "",
      tipoCal: "",
      tipoCalExcel: "",
      ingresoData: "",
      isrData: "",
      resultData: "",
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
      itemsCal: ["Bruto a Neto", "Neto a Bruto", "Excel"],
      itemsCalExcel: ["Bruto a Neto", "Neto a Bruto"],
      dataApi: [],
      itemsFile: [],
      itemsTable: [],
      headers: [],
    };
  },
  methods: {
    getTablasIsrBruto(ingresoData, tablasIsr) {
      if (this.tipoCal == "Bruto a Neto" || this.tipoCal == "Neto a Bruto") {
        ingresoData = this.ingresoData;
      }
      for (let i = 0; i < tablasIsr.data.length; i++) {
        let limInf = tablasIsr.data[i].limInf;
        let porcentaje = tablasIsr.data[i].porcentaje;
        let cuota = tablasIsr.data[i].cuota;
        if (
          tablasIsr.data[i].limInf < ingresoData &&
          tablasIsr.data[i].limSup > ingresoData
        ) {
          let base = ingresoData - limInf;
          let impuestoMar = base * porcentaje;
          this.isrData = impuestoMar + cuota;
          this.isrData = this.isrData.toFixed(2);
          this.resultData = ingresoData - this.isrData;
          this.resultData = this.resultData.toFixed(2);
          this.itemsTable.push({
            Bruto: ingresoData,
            Isr: this.isrData,
            Neto: this.resultData,
          });
        }
      }
    },
    getTablasIsrNeto(ingresoData, tablasIsr) {
      if (this.tipoCal == "Bruto a Neto" || this.tipoCal == "Neto a Bruto") {
        ingresoData = this.ingresoData;
      }
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
                    this.isrData = isrData;
                    this.isrData = this.isrData.toFixed(2);
                    this.resultData = netoFloatProducto;
                    this.resultData = this.resultData.toFixed(2);
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
    subirExcel() {
      let inputFile = document.getElementById("archivo");
      readXlsFile(inputFile.files[0]).then((rows) => {
        this.itemsFile = rows;
        for (let i = 0; i < this.itemsFile.length; i++) {
          this.calculaIngreso(this.itemsFile[i].toString());
        }
        if (this.tipoCalExcel === "Bruto a Neto") {
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
        } else if (this.tipoCalExcel === "Neto a Bruto") {
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
    calculaIngreso(ingresoData) {
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
            this.getTablasIsrBruto(ingresoData, response);
          } else if (this.tipoCal === "Neto a Bruto") {
            this.getTablasIsrNeto(ingresoData, response);
          } else if (this.tipoCal === "Excel") {
            if (this.tipoCalExcel === "Bruto a Neto") {
              this.getTablasIsrBruto(ingresoData, response);
            } else if (this.tipoCalExcel === "Neto a Bruto") {
              this.getTablasIsrNeto(ingresoData, response);
            }
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
  margin-bottom: 27px;
}
</style>
