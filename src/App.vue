<template>
  <v-app>
    <v-main>
      <FormCalculadora />
      <!-- <v-card class="mx-auto" max-width="1200">
        <v-container fluid>
          <v-combobox v-model="vista" :items="itemsCal" label="Tipo Calculadora" outlined dense></v-combobox>
          <v-row dense>
            <v-col v-if="vista == 'Bruto a Neto'" cols="12" sm="12">
              <v-card class="mx-auto" color="green" dark>
                <FormBrutoNeto />
              </v-card>
            </v-col>
            <v-col v-if="vista == 'Neto a Bruto'" cols="12" sm="12">
              <v-card color="#8E250F" dark>
                <FormNetoBruto />
              </v-card>
            </v-col>
            <v-col v-if="vista == 'Excel'" cols="12" sm="12">
              <v-card class="mx-auto" color="#469caa" dark>
                <FormIsrExcel />
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-card> -->
    </v-main>
  </v-app>
</template>

<script>
import FormCalculadora from './components/FormCalculadora.vue';
/* import FormBrutoNeto from "./components/FormBrutoNeto.vue";
import FormNetoBruto from "./components/FormNetoBruto.vue";
import FormIsrExcel from "./components/FormIsrExcel.vue"; */

export default {
  name: "App",
  components: {
    FormCalculadora,
    /* FormBrutoNeto,
    FormNetoBruto,
    FormIsrExcel, */
  },

  data() {
    return {
      vista: "",
      itemsCal: [
        "Bruto a Neto",
        "Neto a Bruto",
        "Excel",
      ],
    };
  },
};
</script>
