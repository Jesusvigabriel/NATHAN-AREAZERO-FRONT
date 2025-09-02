<template>
  <v-container>
    <v-row >
        <v-col align="end">
            <v-btn class="action-button" @click="crearPosicion()">Crear nueva posición</v-btn>
          </v-col>
            <v-col >
            <v-btn class="action-button" @click="imprimirListaEtiquetas()">Imprimir lista etiquetas</v-btn>
        </v-col>
    </v-row>
    <v-row justify="center" >
            <v-col cols="12" md="8" lg="6">
                <v-file-input
                label="Planilla a procesar"
                @change="fileOnChange"
                accept=".xlsx"
                filled
                >
                  <span>Planilla a procesar</span>
                </v-file-input>
            </v-col>
    </v-row>
    <v-row v-show="listaPosiciones.length>0" class="pb-0 mb-0" justify="center">
        <v-col cols="12" md="8" lg="6" class="py-0 my-0"  >
          <v-card-title class="py-1 my-0 justify-center">
              <v-text-field class="search-field"
                  v-model="textoBusqueda"
                  append-icon="mdi-magnify"
                  label="Búsqueda"
                  single-line
                  dense
                  hide-details
              ></v-text-field>
            </v-card-title>
        </v-col>
    </v-row>
    <v-row  justify="center">
      <v-col cols="12" md="8" lg="6" v-if="listaPosiciones.length>0">
        <v-data-table
            :headers="cabecerasPosiciones"
            :items="listaPosiciones"
            :footer-props="{itemsPerPageOptions:[15,30,-1]}"   
            :items-per-page="15" 
            :search="textoBusqueda"
            class="elevation-3" 
        >
        <template v-slot:item.ocupacionPeso="{ item }">
          <v-progress-linear
            :value="(item.pesoOcupadoKg / item.capacidadPesoKg) * 100 || 0"
            :color="getOcupacionColor((item.pesoOcupadoKg / item.capacidadPesoKg) * 100 || 0)"
            height="20"
            striped
          >
            <small>{{ Math.round((item.pesoOcupadoKg / item.capacidadPesoKg) * 100 || 0) }}%</small>
          </v-progress-linear>
        </template>
        <template v-slot:item.ocupacionVolumen="{ item }">
          <v-progress-linear
            :value="(item.volumenOcupadoCm3 / item.capacidadVolumenCm3) * 100 || 0"
            :color="getOcupacionColor((item.volumenOcupadoCm3 / item.capacidadVolumenCm3) * 100 || 0)"
            height="20"
            striped
          >
            <small>{{ Math.round((item.volumenOcupadoCm3 / item.capacidadVolumenCm3) * 100 || 0) }}%</small>
          </v-progress-linear>
        </template>
        <template v-slot:item.Estado="{item}">
          <div class="acciones-flex">
            <v-chip dark color="blue" @click="editarPosicion(item)" class="mx-1"><v-icon>mdi-pencil</v-icon></v-chip>
            <v-chip dark :color="getColorIconoEstado(item)" @click="clickEnVerContenido(item)"><v-icon>mdi-magnify</v-icon></v-chip>
            <v-chip dark color="error" @click="clickEnEliminarPosicion(item)" class="mx-1"><v-icon>mdi-delete-outline</v-icon></v-chip>
            <v-chip dark color="success" @click="imprimirSticker(item)" class="mx-1"><v-icon>mdi-sticker-text-outline</v-icon></v-chip>
          </div>
        </template>
        </v-data-table>
      </v-col>
    </row>

    <v-dialog v-model="mostrarVentanaEdicion" persistent max-width="800px">
      <v-card class="rounded-card">
          <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
          </v-card-title>
          <v-card-text>
              <v-form >
                  <v-container>
                      <v-row>
                          <v-col cols="12" sm="6">
                              <v-text-field label="Nombre" v-model="editedItem.nombre" :rules="[rules.required]"></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6">
                              <v-select
                                  label="Categoría Permitida"
                                  v-model="editedItem.categoriaPermitidaId"
                                  :items="listaCategorias"
                                  item-text="nombre"
                                  item-value="id"
                                  clearable
                              ></v-select>
                          </v-col>
                      </v-row>
                      <v-row>
                          <v-col cols="12" sm="6" md="3">
                              <v-text-field type="number" label="Capacidad Peso (Kg)" v-model.number="editedItem.capacidadPesoKg"></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="3">
                              <v-text-field type="number" label="Capacidad Volumen (cm³)" v-model.number="editedItem.capacidadVolumenCm3"></v-text-field>
                          </v-col>
                           <v-col cols="12" sm="6" md="3">
                              <v-text-field type="number" label="Factor Desperdicio (%)" v-model.number="editedItem.factorDesperdicio"></v-text-field>
                          </v-col>
                      </v-row>
                      <v-row>
                          <v-col cols="12" sm="6" md="4">
                              <v-text-field type="number" label="Coordenada X" v-model.number="editedItem.coordX"></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="4">
                              <v-text-field type="number" label="Coordenada Y" v-model.number="editedItem.coordY"></v-text-field>
                          </v-col>
                      </v-row>
                  </v-container>
              </v-form>
          </v-card-text>
          <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="cancelarEdicion()">Cancelar</v-btn>
              <v-btn color="blue darken-1" text @click="confirmarEdicion()">Guardar</v-btn>
          </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="mostrarDetallePosiciones" persistent max-width="1000px">
      <v-card class="rounded-card">
          <v-card-title>
              <span class="text-h5">Contenido de la posición {{detallePosicionesPosicion}}</span>
          </v-card-title>
          <v-col class="py-0" v-if="detallePosicionesPosicion=='Area54-Nave1' || detallePosicionesPosicion=='zonacero' || detallePosicionesPosicion=='Itsa' ">
            <v-btn @click="actualizaFechaPosicion(detallePosicionesContenido)"  block color="blue" dark>Actualizar Fecha</v-btn>
          </v-col>
          <v-spacer></v-spacer>
          <v-card-text>
            <v-data-table 
                :headers="cabecerasDetallePosiciones" 
                :items="detallePosicionesContenido"
                class="elevation-3" 
            >
              <template v-slot:item.Acciones="{item}">
                <v-chip dark color="error" @click="clickEnVaciarItemDePosicion(item)"><v-icon>mdi-delete-outline</v-icon></v-chip>
              </template>
            </v-data-table>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue" text @click="exportarContenidoAExcel()">Exportar a Excel</v-btn>
            <v-btn color="success" text @click="cerrarDetallePosiciones()">Cerrar</v-btn>
          </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="mostrarResultadoCreacionMasiva" persistent max-width="1000px">
      <v-card class="rounded-card">
          <v-card-title>
              <span class="text-h5">Proceso finalizado</span>
          </v-card-title>
          <v-spacer></v-spacer>
          <v-card-text>
            <v-data-table 
                :headers="cabecerasPosicionesCreadas" 
                :items="listaPosicionesExcel"
                class="elevation-3" 
            >
            </v-data-table>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="success" text @click="cerrarResultadoPosiciones()">Cerrar</v-btn>
          </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>

import store from '@/store'
// import posiciones from '@/store/posiciones' // Old API
import posicion from '@/store/posicionesV3' // New API (aliased as posicion)
import productos from '@/store/productosV3'
import {xlsx, read, utils} from 'xlsx'
import excel from "exceljs"
import {saveAs} from "file-saver"
import { ExcelIcon } from '@/components/icons'

export default {
  name: "VerPosiciones",

  data() {
    return {
      mostrarDetallePosiciones: false,
      mostrarResultadoCreacionMasiva: false,
      detallePosicionesPosicion: '',
      detallePosicionesContenido: [],
      listaPosicionesExcel: [],
      posicionSeleccionada: {},
      mostrarVentanaEdicion: false,
      listaPosiciones: [],
      listaCategorias: [],
      editedIndex: -1,
      editedItem: {
        nombre: '',
        capacidadPesoKg: 0,
        capacidadVolumenCm3: 0,
        factorDesperdicio: 0,
        categoriaPermitidaId: null,
        coordX: 0,
        coordY: 0,
      },
      defaultItem: {
        nombre: '',
        capacidadPesoKg: 0,
        capacidadVolumenCm3: 0,
        factorDesperdicio: 0,
        categoriaPermitidaId: null,
        coordX: 0,
        coordY: 0,
      },
      rules: {
        required: value => !!value || 'Este campo es requerido.',
      },
      cabecerasPosiciones: [
          {text: 'Posición', value: 'nombre', sortable: true},
          {text: 'Ocupación Peso', value: 'ocupacionPeso', sortable: false, width: '150px'},
          {text: 'Ocupación Volumen', value: 'ocupacionVolumen', sortable: false, width: '150px'},
          {text: 'Fecha inventario', value: 'FechaInventario', sortable: true},
          {text: 'Usuario', value: 'UsuarioInventario', sortable: true},
          {text: '', value: 'Estado', sortable: false, width: '200px'},
      ],
      cabecerasDetallePosiciones: [
          {text: 'Id', value: 'IdProducto'},
          {text: 'Nombre', value: 'NombreProducto'},
          {text: 'Barcode', value: 'BarcodeProducto'},
          {text: 'Unidades', value: 'Unidades'},
          {text: 'Fecha', value: 'Fecha'},
          {text: '', value: 'Acciones'},
      ],
      cabecerasPosicionesCreadas: [
        {text: 'Posicion', value: 'posicion'},
        {text: 'Estado', value: 'estado'},
      ],      
      textoBusqueda: '',
      fechaMayor: null
    }
  },

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Crear Nueva Posición' : 'Editar Posición'
    },
  },

  methods: {
    async fetchCategorias() {
      try {
        this.listaCategorias = await posicion.categorias.getAll();
      } catch (error) {
        this.mostrarError('No se pudieron cargar las categorías.');
      }
    },

    actualizaFechaPosicion(items){
      this.fechaMayor = '2030-01-01 16:11:11'
      this.editFechaPosicion(items[0].posicionId,this.fechaMayor)
    },

    clickEnVaciarItemDePosicion(item) {
      const textoPrimario="Si, desposicionar el item"
      const textoSecundario="Cancelar"
      const ad={
        titulo: 'Confirma el desposicionado del item seleccionado?',
        mensaje: 'Este proceso es <b>irreversible</b>;;Está <u>completamente seguro?</u>',
        botonPrimario: textoPrimario,
        botonSecundario: textoSecundario,
        callback: ((respuesta) => {
          if (respuesta===textoPrimario) {
            this.VaciarItemDePosicionConfirmado(this.posicionSeleccionada, item)
          }
        })
      }
      store.dispatch("alertDialog/mostrar", ad)
    },
    async VaciarItemDePosicionConfirmado(posicionSeleccionada, item) {
      try {
        await productos.registrarDesposicionamiento(item.IdProducto, posicionSeleccionada.Id, item.Unidades)
      } catch (error) {
        this.mostrarError(error)
      }
      this.mostrarDetallePosiciones=false
      await this.popularListaPosiciones()
    },    
    cerrarDetallePosiciones() {
      this.mostrarDetallePosiciones=false
      this.detallePosicionesContenido=[]
    },    
    cerrarResultadoPosiciones(){
      this.mostrarResultadoCreacionMasiva=false
      this.listaPosicionesExcel=[]
    },

    clickEnEliminarPosicion(item) {
      const textoPrimario="Si, borrarla"
      const textoSecundario="No borrarla"
      const ad={
        titulo: 'Confirma la eliminación de la posición seleccionada?',
        mensaje: 'Este proceso es <b>irreversible</b>;;Está <u>completamente seguro?</u>',
        botonPrimario: textoPrimario,
        botonSecundario: textoSecundario,
        callback: ((respuesta) => {
          if (respuesta===textoPrimario) {
            this.eliminarPosicion(item)
          }
        })
      }
      store.dispatch("alertDialog/mostrar", ad)
    },
    async eliminarPosicion(item) {
      try {
        const contenidoPosicion = await posicion.getContent(item.Id)
        if(contenidoPosicion.length > 0){
          this.mostrarMensaje("Error", "Debe vaciar la posición antes de eliminarla.")
        }else{
          await posicion.eliminar(item.Id) // Assuming `posicion` has `eliminar`
          this.popularListaPosiciones()
        }
      } catch (error) {
        this.mostrarError(error)
      }
    },

    crearPosicion() {
      this.editedItem = Object.assign({}, this.defaultItem)
      this.editedIndex = -1
      this.mostrarVentanaEdicion = true
    },

    editarPosicion(item) {
      this.editedIndex = this.listaPosiciones.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.mostrarVentanaEdicion = true
    },

    cancelarEdicion() {
      this.mostrarVentanaEdicion = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    async confirmarEdicion() {
      if (this.editedItem.nombre) {
        try {
          if (this.editedIndex > -1) {
            // Update
            await posicion.updatePosicion(this.editedItem.Id, this.editedItem)
          } else {
            // Create
            await posicion.createPosicion(this.editedItem)
          }
          this.popularListaPosiciones()
        } catch (error) {
          this.mostrarError(error)
        }
        this.cancelarEdicion()
      } else {
        this.mostrarMensaje("Error", "El nombre de la nueva posición no puede estar vacío")
      }
    },

    async clickEnVerContenido(item) {
      try {
        const result=await posicion.getContent(item.Id)
        this.detallePosicionesContenido = result.map(el => ({
          ...el,
          Fecha: el.Fecha ? new Date(el.Fecha).toLocaleDateString() : ""
        }));

        if (result.length === 0) {
           this.mostrarMensaje("Información", "La posición está vacía.")
        }

        this.detallePosicionesPosicion=item.Nombre
        this.posicionSeleccionada=item
        this.mostrarDetallePosiciones=true

      } catch (error) {
        this.mostrarError(error)
      }
    },

    editFechaPosicion(id,fecha){
      posicion.editFechaPos_Prod(id,fecha)
                .then(() => {
                    // console.log("Respuesta", respuesta);
                })
                .catch(puteada => {
                    this.mostrarError(puteada)
                })
    },

    async exportarContenidoAExcel() {
        const item = this.posicionSeleccionada
        const result=await posicion.getContent(item.Id)

        if (result.length>0) {
            const workbook=new excel.Workbook()
            const worksheet=workbook.addWorksheet(item.Nombre)

            worksheet.getRow(1).values=[`Detalle de la posición ${item.Nombre}`]
            
            worksheet.columns=[
                {width: 15}, 
                {width: 30},
                {width: 50},
                {width: 15},
            ]

            let renglon=3
            worksheet.getRow(renglon).values=['Artículo', 'Barcode', 'Nombre', 'Unidades']
            worksheet.views = [{state: 'frozen', ySplit: renglon}]
            worksheet.autoFilter = `A${renglon}:D${renglon}`

            result.forEach(e => {
                renglon++
                worksheet.getRow(renglon).values=[
                    e.IdProducto,
                    e.BarcodeProducto,
                    e.NombreProducto,
                    e.Unidades,
                ]
            })

            renglon++
            const columnasASumar=["D"]
            for (const unaColumna of columnasASumar) {
                const celdaSuma=worksheet.getCell(`${unaColumna}${renglon}`)
                celdaSuma.value={formula: `SUM(${unaColumna}1:${unaColumna}${renglon-1})`}
                celdaSuma.font={bold: true}
           }

          worksheet.eachRow ( (row, rowNumber) => {
              row.eachCell ( (cell, colNumber) => {
                  if (rowNumber===1 || rowNumber==renglon) {
                      cell.font={size: 16, bold: true}
                  } else {
                      cell.font={size: 14}
                  }
              })
          } )

          const buf=await workbook.xlsx.writeBuffer()
          saveAs(new Blob([buf]), `Detalle_Posicion_${item.Nombre}.xlsx`)
        }
    },
    fileOnChange(archivoLeido) {
            if (archivoLeido!=null) {
                const lector=new FileReader()
                lector.onload = ev => {
                const datosCrudos = ev.target.result;
                const planillaCruda = read(datosCrudos, {type: "binary", cellDates: true, cellNF: true, cellText:true})
                const nombreHoja1=planillaCruda.SheetNames[0]
                const datosPlanilla=utils.sheet_to_json(planillaCruda.Sheets[nombreHoja1])
                this.verificarColumnasExcel(datosPlanilla)
                }

                lector.readAsBinaryString(archivoLeido)            
            }
    },
    verificarColumnasExcel(planilla) {
    let columnasObligatorias= (['Posicion'])
    let columnasFaltantes=[]
    let filaActual=1
    let tituloMostrado=false
    let mensaje=''
    planilla.forEach(unaFila => {
        filaActual++
        columnasObligatorias.forEach(unaColumnaObligatoria => {
        if (!Object.keys(unaFila).includes(unaColumnaObligatoria)) {
            if (!columnasFaltantes.includes(unaColumnaObligatoria)) {
                if (!tituloMostrado) {
                    mensaje = "Faltan las siguientes columnas:"
                    tituloMostrado=true
                }
                
                if(!filaActual){
                  mensaje += ";Fila: "+filaActual+" - Columna: "+unaColumnaObligatoria
                }
            }
        }
        })
      })
        if (tituloMostrado) {
            store.dispatch("alertDialog/mostrar", {titulo: "No se puede procesar", mensaje})
        } else {
            this.procesarPlanilla(planilla)
        }
    },
    procesarPlanilla(planilla){
      const textoPrimario="Aceptar"
      const textoSecundario="Cancelar"
      const ordenDetalle={
          titulo: `Guardar ${parseInt(planilla.length)} posiciones`,
          mensaje: `continuar?`,
          botonSecundario: textoSecundario,
          botonPrimario: textoPrimario,
          callback: ((respuesta) => {
              if (respuesta===textoPrimario) {
                  for(const fila of planilla){
                    try {
                      posicion.createPosicion({ nombre: fila.Posicion.trim() })
                      .then(result => {
                        this.listaPosicionesExcel.push({
                          posicion: result.nombre,
                          estado: "Guardado"
                        })
                      })
                      .catch (error => {
                        this.listaPosicionesExcel.push({
                          posicion: fila.Posicion,
                          estado: "Error: " + error
                        })
                      })
                    } catch (error) {
                      this.mostrarMensaje("Ha ocurrido un error", error)
                    }
                  }
                  this.popularListaPosiciones()
                  this.mostrarResultadoCreacionMasiva = true
              }
          })
      }
      store.dispatch("alertDialog/mostrar",  ordenDetalle) 
    },

    imprimirListaEtiquetas(){
        const listaImprimir = this.listaPosiciones.filter(e => {
        const searchText = this.textoBusqueda.toUpperCase();
        return e.Nombre.toUpperCase().includes(searchText);
      })
      
      if(listaImprimir.length != 0){
        const textoPrimario="Aceptar"
        const textoSecundario="Cancelar"
        const ordenDetalle={
            titulo: `Esta por imprimir ${parseInt(listaImprimir.length)} posiciones`,
            mensaje: `Esto puede tardar dependiendo de la cantidad, continuar?`,
            botonSecundario: textoSecundario,
            botonPrimario: textoPrimario,
            callback: ((respuesta) => {
                if (respuesta===textoPrimario) {
                  posicion.imprimirListaStickers(listaImprimir)
                }
            })
        }
        store.dispatch("alertDialog/mostrar",  ordenDetalle) 
      }else{
        this.mostrarMensaje("ERROR", "Esta intentando imprimir 0 etiquetas")
      }
    },
    getColorIconoEstado(item) {
      if (item.FechaInventario!=null && item.FechaInventario!="") {
        return "success"
      } else {
        return 'warning'
      }
    },
    getOcupacionColor(percentage) {
      if (percentage < 50) return 'green';
      if (percentage < 85) return 'orange';
      return 'red';
    },

    getIconoEstado() {
      return 'mdi-magnify'
    },
    async popularListaPosiciones() {
      try {
        const response = await posicion.getPosicionesConCapacidad()
        this.listaPosiciones = response.map(e => ({
          ...e,
          FechaInventario: (e.FechaInventario!==null && e.FechaInventario!=="") ? new Date(e.FechaInventario).toLocaleDateString() : ""
        }));
      } catch (error) {
        this.mostrarError(error)
      }
    },
    mostrarMensaje(titulo, mensaje) {
        store.dispatch("alertDialog/mostrar", {titulo, mensaje})
    },
    mostrarError(mensaje) {
      store.dispatch("snackbar/mostrar", mensaje)
    },
    sonidoExito() {
      store.dispatch('sonidos/exito')
    },
    async imprimirSticker(item) {
      posicion.imprimirSticker(item)
    }
  },

  components: {
    ExcelIcon
  },

  created() {
    store.dispatch('actualizarTituloPrincipal', 'Ver posiciones')
    this.popularListaPosiciones()
    this.fetchCategorias()
  }

}
</script>

<style scoped>
.acciones-flex {
  display: flex;
  gap: 8px;
  justify-content: center;
  align-items: center;
}
</style>