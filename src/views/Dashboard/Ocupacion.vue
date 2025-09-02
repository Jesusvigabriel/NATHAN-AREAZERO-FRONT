<template>
  <v-container>
    <v-row>
      <v-col>
        <h1 class="text-h4">Dashboard de Ocupación</h1>
        <p>Análisis de la capacidad utilizada en el almacén.</p>
      </v-col>
    </v-row>

    <!-- Filters -->
    <v-row>
      <v-col cols="12" md="4">
        <v-select
          v-model="selectedEmpresa"
          :items="listaEmpresas"
          item-text="Nombre"
          item-value="Id"
          label="Filtrar por Empresa"
          clearable
          @change="fetchOcupacionData"
        ></v-select>
      </v-col>
      <v-col cols="12" md="4">
        <!-- Assuming zones are just typed in for now, can be a select if an endpoint exists -->
        <v-text-field
          v-model="selectedZona"
          label="Filtrar por Zona"
          clearable
          @change="fetchOcupacionData"
        ></v-text-field>
      </v-col>
    </v-row>

    <!-- Charts -->
    <v-row v-if="!loading && chartData">
      <v-col cols="12" md="6">
        <v-card>
          <v-card-title>Ocupación por Peso (Kg)</v-card-title>
          <v-card-text>
            <bar-chart v-if="chartData.peso" :chart-data="chartData.peso"></bar-chart>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="6">
        <v-card>
          <v-card-title>Ocupación por Volumen (cm³)</v-card-title>
          <v-card-text>
             <bar-chart v-if="chartData.volumen" :chart-data="chartData.volumen"></bar-chart>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
     <v-row v-if="loading">
        <v-col class="text-center">
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
            <p>Cargando datos...</p>
        </v-col>
     </v-row>
  </v-container>
</template>

<script>
import { Bar } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
import posicionV3 from '@/store/posicionesV3'
import empresas from "@/store/empresasV3"

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

export default {
  name: 'OcupacionDashboard',
  components: {
    'bar-chart': Bar
  },
  data() {
    return {
      loading: false,
      chartData: null,
      listaEmpresas: [],
      selectedEmpresa: null,
      selectedZona: '',
    }
  },
  async created() {
    await this.fetchEmpresas();
    await this.fetchOcupacionData();
  },
  methods: {
    async fetchEmpresas() {
        try {
            this.listaEmpresas = await empresas.getAll(true);
        } catch (error) {
            console.error("Error fetching companies:", error);
            // Handle error (e.g., show a notification)
        }
    },
    async fetchOcupacionData() {
      this.loading = true;
      try {
        const data = await posicionV3.getOcupacion(this.selectedEmpresa, this.selectedZona);

        const pesoData = {
          labels: ['Ocupado', 'Disponible'],
          datasets: [{
            label: 'Peso (Kg)',
            backgroundColor: ['#f87979', '#a1e8a1'],
            data: [data.pesoOcupadoKg, data.capacidadPesoKg - data.pesoOcupadoKg]
          }]
        };

        const volumenData = {
          labels: ['Ocupado', 'Disponible'],
          datasets: [{
            label: 'Volumen (cm³)',
            backgroundColor: ['#79a1f8', '#f8d479'],
            data: [data.volumenOcupadoCm3, data.capacidadVolumenCm3 - data.volumenOcupadoCm3]
          }]
        };

        this.chartData = {
            peso: pesoData,
            volumen: volumenData
        };

      } catch (error) {
        console.error("Error fetching occupancy data:", error);
        // Handle error
      } finally {
        this.loading = false;
      }
    }
  }
}
</script>

<style scoped>
.v-card {
  margin-bottom: 16px;
}
</style>
