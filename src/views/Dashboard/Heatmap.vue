<template>
  <v-container>
    <v-row>
      <v-col>
        <h1 class="text-h4">Mapa de Calor del Almacén</h1>
        <p>Visualización de la intensidad de uso de las posiciones.</p>
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
          @change="fetchHeatmapData"
        ></v-select>
      </v-col>
      <v-col cols="12" md="4">
        <v-text-field
          v-model="selectedPeriodo"
          label="Periodo (YYYY-MM)"
          placeholder="Ej: 2023-10"
          clearable
          @change="fetchHeatmapData"
        ></v-text-field>
      </v-col>
      <v-col cols="12" md="4">
        <v-text-field
          v-model="selectedZona"
          label="Filtrar por Zona"
          clearable
          @change="fetchHeatmapData"
        ></v-text-field>
      </v-col>
    </v-row>

    <!-- Heatmap -->
    <v-row v-if="!loading">
      <v-col>
        <v-card>
          <v-card-title>Intensidad de Movimientos</v-card-title>
          <v-card-text>
            <div class="heatmap-grid" :style="gridStyle">
              <div
                v-for="cell in heatmapCells"
                :key="`${cell.coordX}-${cell.coordY}`"
                class="heatmap-cell"
                :style="{ backgroundColor: cell.color }"
              >
                <v-tooltip top>
                  <template v-slot:activator="{ on }">
                    <span v-on="on">{{ cell.intensity.toFixed(2) }}</span>
                  </template>
                  <span>Pos: ({{ cell.coordX }}, {{ cell.coordY }})</span>
                </v-tooltip>
              </div>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
     <v-row v-if="loading">
        <v-col class="text-center">
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
            <p>Cargando datos del mapa de calor...</p>
        </v-col>
     </v-row>
  </v-container>
</template>

<script>
import posicionV3 from '@/store/posicionesV3';
import empresas from "@/store/empresasV3";

export default {
  name: 'HeatmapDashboard',
  data() {
    return {
      loading: false,
      heatmapData: [],
      listaEmpresas: [],
      selectedEmpresa: null,
      selectedPeriodo: new Date().toISOString().slice(0, 7), // Default to current month
      selectedZona: '',
      maxX: 0,
      maxY: 0,
    };
  },
  computed: {
    gridStyle() {
      return {
        'grid-template-columns': `repeat(${this.maxX + 1}, 1fr)`,
      };
    },
    heatmapCells() {
      const cells = [];
      const intensityMap = new Map(this.heatmapData.map(d => [`${d.coordX}-${d.coordY}`, d.intensidad]));
      const maxIntensity = Math.max(...this.heatmapData.map(d => d.intensidad), 0);

      for (let y = 0; y <= this.maxY; y++) {
        for (let x = 0; x <= this.maxX; x++) {
          const key = `${x}-${y}`;
          const intensity = intensityMap.get(key) || 0;
          cells.push({
            coordX: x,
            coordY: y,
            intensity: intensity,
            color: this.getColorForIntensity(intensity, maxIntensity),
          });
        }
      }
      return cells;
    },
  },
  async created() {
    await this.fetchEmpresas();
    await this.fetchHeatmapData();
  },
  methods: {
    async fetchEmpresas() {
        try {
            this.listaEmpresas = await empresas.getAll(true);
        } catch (error) {
            console.error("Error fetching companies:", error);
        }
    },
    async fetchHeatmapData() {
      if (!this.selectedEmpresa) return;
      this.loading = true;
      try {
        const data = await posicionV3.getHeatmap(this.selectedEmpresa, this.selectedPeriodo, this.selectedZona);
        this.heatmapData = data;
        this.maxX = Math.max(...data.map(d => d.coordX), 0);
        this.maxY = Math.max(...data.map(d => d.coordY), 0);
      } catch (error) {
        console.error("Error fetching heatmap data:", error);
      } finally {
        this.loading = false;
      }
    },
    getColorForIntensity(intensity, maxIntensity) {
      if (intensity === 0) return '#f0f0f0'; // Neutral color for zero intensity
      const ratio = maxIntensity > 0 ? intensity / maxIntensity : 0;
      // Simple green to red gradient
      const red = Math.round(255 * ratio);
      const green = Math.round(255 * (1 - ratio));
      return `rgb(${red}, ${green}, 0)`;
    },
  },
};
</script>

<style scoped>
.heatmap-grid {
  display: grid;
  gap: 2px;
  border: 1px solid #ccc;
  padding: 5px;
  background-color: #e0e0e0;
}
.heatmap-cell {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 0.7rem;
  font-weight: bold;
  text-shadow: 1px 1px 2px black;
  border-radius: 3px;
}
</style>
