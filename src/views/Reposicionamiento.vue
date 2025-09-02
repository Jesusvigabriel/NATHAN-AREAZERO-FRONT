<template>
    <v-container>
        <v-row justify="center">
            <v-col align="center" cols="12" md="6">
                <v-alert
                    v-model="alertaMensajePresentacion"
                    color="deep-purple accent-5"
                    dark
                    dismissible
                >
                    <h3 class="capitalize-words">Reposicionamiento de stock</h3>
                    Esta opción debe ser utilizada <b><u>únicamente</u></b> para mover unidades de una ubicación previamente asignada a otra.<br>
                    <b><u>No debe ser utilizada para posicionar unidades</u></b>, es decir, para asignarle ubicación a unidades que se encontraban sin ubicación asignada.<br>
                    Si desea <b><u>asignarle posición a unidades que no tenían ubicación previa</u></b> <v-btn color="green" @click="clickEnPosicionamiento" >ir a <b>👉 Posicionamiento 👈</b></v-btn>
                </v-alert>            
            </v-col>
        </v-row>
        <v-row justify="center">
            <v-col cols="12" md="6">
                <v-select 
                    v-model="idEmpresa" 
                    dense 
                    prepend-inner-icon="mdi-factory"
                    @change="eligioEmpresa" 
                    :chips="false" 
                    :items="listaEmpresas" 
                    item-value="Id" 
                    item-text="Nombre" 
                    label="Seleccione empresa"
                    class="my-0 py-0"
                    id="SelectorEmpresa">
                </v-select>
            </v-col>
        </v-row>
        <v-row v-if="idEmpresa != null">
            <v-col cols="6">
                <v-file-input 
                label="Planilla a procesar" 
                @change="fileOnChange" 
                accept=".xlsx"
                filled
                prepend-icon="mdi-microsoft-excel"
                >Importar planilla excel</v-file-input>
            </v-col>
            <v-col class="col-sm-12 col-md-6">
                <v-btn color="blue v-btn--block" dark @click="descargarExcelEjemplo()">Descargar excel de ejemplo<v-icon>mdi-microsoft-excel</v-icon></v-btn>
            </v-col>
        </v-row>
        <v-row justify="center" >
            <v-col cols="5" md="2" v-if="barcodePosicionOrigen.mostrar">
                <v-text-field 
                    ref="barcodePosicionOrigen"
                    inputmode="none" 
                    prepend-inner-icon="mdi-bookshelf" 
                    @keypress.enter="procesarBarcodePosicionOrigen" 
                    label="Barcode posicicion origen" 
                    v-model="barcodePosicionOrigen.dato" 
                    dense>
                </v-text-field>
            </v-col>
            <v-col cols="1" class="mx-0">
                <v-icon color="green" v-show="barcodePosicionOrigen.correcto">mdi-checkbox-marked-circle-outline</v-icon>
                <v-icon color="red" v-show="barcodePosicionOrigen.incorrecto">mdi-alert-circle-outline</v-icon>
            </v-col>
            <v-col cols="5" md="2" v-if="barcodePosicionDestino.mostrar">
                <v-text-field 
                    ref="barcodePosicionDestino"
                    inputmode="none" 
                    prepend-inner-icon="mdi-bookshelf" 
                    @keypress.enter="procesarBarcodePosicionDestino" 
                    label="Barcode posicicion destino" 
                    v-model="barcodePosicionDestino.dato" 
                    class="mx-0 px-0"
                    dense>
                </v-text-field>
            </v-col>
            <v-col cols="1" class="mx-0 px-0">
                <v-icon color="green" v-show="barcodePosicionDestino.correcto">mdi-checkbox-marked-circle-outline</v-icon>
                <v-icon color="red" v-show="barcodePosicionDestino.incorrecto">mdi-alert-circle-outline</v-icon>
            </v-col>
             <v-col cols="12" md="4" v-if="posicionDestinoOcupacion">
                 <v-alert dense outlined type="info">
                    Disponible:
                    <strong>{{ (posicionDestinoOcupacion.capacidadPesoKg - posicionDestinoOcupacion.pesoOcupadoKg).toFixed(2) }} Kg</strong>,
                    <strong>{{ (posicionDestinoOcupacion.capacidadVolumenCm3 - posicionDestinoOcupacion.volumenOcupadoCm3).toFixed(2) }} cm³</strong>
                </v-alert>
            </v-col>
        </v-row>
        <v-row justify="center" v-if="barcodePosicionOrigen.correcto" class="my-0 py-0">
            <v-col cols="12" md="6" class="my-0 py-0">
                <v-text-field 
                    ref="barcodeArticulo" 
                    inputmode="none" 
                    prepend-inner-icon="mdi-barcode-scan" 
                    @keypress.enter="procesarBarcodeArticulo" 
                    label="Barcode artículo" 
                    v-model="barcodeArticulo" 
                    dense 
                    class="my-0 py-0"
                    >
                </v-text-field>
            </v-col>
            <v-col cols="5" md="2" v-if="cantidadArticulos.mostrar" class="my-0 py-0">
                <v-text-field 
                    ref="cantidad" 
                    type="number"   
                    prepend-inner-icon="mdi-calculator"  
                    @keypress.enter="procesarCantidad" 
                    label="Cantidad"  
                    v-model="cantidad"
                    :error-messages="capacityErrorMessages"
                    dense 
                    id="CantidadAIngresar" 
                    class="my-0 py-0"
                    >
                </v-text-field>
            </v-col>
        </v-row>
        <v-row justify="center" v-if="barcodePosicionDestino.correcto" class="my-0 py-0">
            <v-col align="center" cols="12" md="3" class="my-0 py-0">
                <v-checkbox label="La posición queda vacía" v-model="posicionQuedaVacia" class="my-0 py-0"></v-checkbox>
            </v-col>
            <v-col cols="12" md="3" class="my-0 py-0">
                <v-btn color="info" block @click="clickEnProcesar()">Confirmar el procesamiento</v-btn>
            </v-col>
        </v-row>
        <v-row v-for="(item, index) in listaBarcodesArticulos" :key="index" justify="center">
            <v-col cols="12" md="6" class="py-0 my-0" align="center">
                <v-alert color="yellow" class="py-0 my-0">{{item[0]+" Cant."+item[1]}}
                    <v-icon color="red" @click="eliminarLectura(item, index)" >mdi-delete-outline</v-icon>
                </v-alert>
            </v-col>
        </v-row>
        <v-row >
            <v-col>
                <v-alert v-if="!tieneLOTE" dark class="my-1 mx-0" v-for="(item, index) in listaCodigosFallidos" :key="index" dense :color="colorLectura(item)" >{{item.Barcode}} - {{item.Descripcion}}</v-alert>
                <v-alert v-if="tieneLOTE" dark class="my-1 mx-0" v-for="(item, index) in listaCodigosFallidos" :key="index" dense :color="colorLectura(item)" >{{"Lote: " + item.Lote}} - {{"Posicion: " + item.Posicion}} - {{"IdProducto:" + item.IdProducto}} - {{ " " + item.Mensaje}}</v-alert>
            </v-col>
        </v-row>
        <v-row v-if="listaBarcodesError.length>0" >
            <v-col class="py-0 my-0">
                <v-alert color="red" class="py-0 my-1" dark><h3 class='capitalize-words'>Barcodes inexistentes</h3></v-alert>
            </v-col>
        </v-row>
        <v-row v-for="(item, index) in listaBarcodesError" :key="index">
            <v-col class="py-0 my-0">
                <v-alert color="red" class="py-0 my-0" dark>{{item}}</v-alert>
            </v-col>
        </v-row>
        
        <v-row >
            <v-col>
                <v-alert v-if="!tieneLOTE" dark class="my-1 mx-0" v-for="(item, index) in listaCodigosLeidos" :key="index" dense :color="colorLectura(item)" >{{item.Barcode}} - {{item.Descripcion}}</v-alert>
                <v-alert v-if="tieneLOTE" dark class="my-1 mx-0" v-for="(item, index) in listaCodigosLeidos" :key="index" dense :color="colorLectura(item)" >{{"Lote:" + item.Lote}} - {{"IdProducto:" + item.IdProducto}} - {{"Unidades:" + item.Unidades}} - {{"Posicion:" + item.Posicion}}</v-alert>
            </v-col>
        </v-row>
    </v-container>
</template>


<script>

import empresas from "@/store/empresasV3"
import productos from "@/store/productosV3"
import posiciones from "@/store/posiciones"
import store from "@/store"
import router from '../router'
import {xlsx, read, utils} from 'xlsx'
import excel from "exceljs"
import {saveAs} from "file-saver"
import posicionesV3 from "@/store/posicionesV3"

export default {
    name: "ReposicionamientoConVaciado",

    data() {
        return {
            alertaMensajePresentacion: true,
            idEmpresa: null,
            listaEmpresas: [],
            posicionOrigen: null,
            posicionDestino: null,
            barcodePosicionOrigen: {dato: process.env.NODE_ENV=='development' ? '001-01-01' : '', mostrar: false, correcto: false, incorrecto: false},
            barcodePosicionDestino: {dato: process.env.NODE_ENV=='development' ? '001-01-02' : '', mostrar: false, correcto: false, incorrecto: false},
            cantidadArticulos: {mostrar: false, correcto: false, incorrecto: false},
            listaArticulosOK: [],
            listaBarcodesArticulos: [],
            listaBarcodesError: [],
            barcodeArticulo: process.env.NODE_ENV=='development' ? 'T01' : '',
            cantidad:0,
            posicionQuedaVacia: false,
            reposicionStockUnitario: false,
            manejaStockUnitario:false,
            tieneLOTE: false,
            listaCodigosLeidos: [],
            listaCodigosFallidos: [],
            userName: null,
            tipoCodigo:null,
            posicionDestinoOcupacion: null,
            articuloActual: null,
        }
    },

    computed: {
        capacityErrorMessages() {
            if (!this.articuloActual || !this.posicionDestinoOcupacion || !this.cantidad) {
                return [];
            }

            const errors = [];
            const requiredWeight = (this.articuloActual.pesoKg || 0) * this.cantidad;
            const requiredVolume = (this.articuloActual.volumenCm3 || 0) * this.cantidad;
            const availableWeight = this.posicionDestinoOcupacion.capacidadPesoKg - this.posicionDestinoOcupacion.pesoOcupadoKg;
            const availableVolume = this.posicionDestinoOcupacion.capacidadVolumenCm3 - this.posicionDestinoOcupacion.volumenOcupadoCm3;

            if (requiredWeight > availableWeight) {
                errors.push(`Peso excede la capacidad (${requiredWeight.toFixed(2)} > ${availableWeight.toFixed(2)} Kg)`);
            }
            if (requiredVolume > availableVolume) {
                errors.push(`Volumen excede la capacidad (${requiredVolume.toFixed(2)} > ${availableVolume.toFixed(2)} cm³)`);
            }

            return errors;
        }
    },

    methods: {
        clickEnPosicionamiento() {
            router.push("/PosicionamientoStock")
        },
        colorLectura: function(item) {
            return item.Correcto ? "green" : "red";
        },
        clickEnProcesar() {
            const pregunta={
                titulo: "Confirma?", 
                mensaje: "Confirma el procesamiento "+ (this.posicionQuedaVacia ? "<b>CON</b>" : "<b>SIN</b>") + " vaciamiento de posición?",
                botonPrimario: "Procesar", 
                botonSecundario: "Cancelar", 
                callback: respuesta => {
                    if (respuesta=="Procesar") {
                        this.verificarBarcodesArticulos()
                    }
                }}
            store.dispatch("alertDialog/mostrar", pregunta)
        },
        async verificarBarcodesArticulos() {
            let todosExisten=true
            
            //si tiene lote no hace nada porque ya fueron verificados cuando se leyo el barcode
            if(!this.tieneLOTE){
                for (const unArticulo of this.listaBarcodesArticulos) {
                    await productos.getByBarcodeAndEmpresa(unArticulo[0], this.idEmpresa)
                        .then(respuesta => {
                            respuesta.cantidad= unArticulo[1]
                            this.listaArticulosOK.push(respuesta)
                        })
                        .catch(error => {
                            this.listaCodigosLeidos.push({Barcode: unArticulo[0], Correcto: false, Descripcion: error})
                        })
                }
            }
            if (this.listaArticulosOK.length > 0) {
                this.procesarArticulos(this.listaArticulosOK)
            }
            this.listaBarcodesArticulos=[]
        },
        procesarArticulos(articulos) {
            if(this.tieneLOTE){
                const payload={
                    vaciarPosicionOrigen: this.posicionQuedaVacia, 
                    articulos,
                    posicionOrigen: this.barcodePosicionOrigen.dato,
                    posicionDestino: this.barcodePosicionDestino.dato,
                    usuario: this.userName,
                }
                productos.registrarReposicionamiento(payload)
                .then(respuesta => {
                    for(const resultados of respuesta){
                        if(resultados.unProducto != undefined){
                            if(resultados.result.status == "OK"){
                                this.listaCodigosLeidos.unshift({Lote: resultados.unProducto.lote, Correcto: true, IdProducto: resultados.unProducto.Id, Unidades: resultados.unProducto.cantidad, Posicion: this.posicionDestino.Nombre})
                            }else{
                                this.listaCodigosLeidos.unshift({Lote: resultados.unProducto.lote, Correcto: false, IdProducto: resultados.unProducto.Id, Unidades: resultados.unProducto.cantidad, Posicion: this.posicionDestino.Nombre})
                            }
                        }
                    }
                    const mensajeExito={titulo: "El proceso a finalizado", mensaje: " "}
                    store.dispatch("alertDialog/mostrar", mensajeExito)
                    Object.assign(this.barcodePosicionOrigen, {dato: "", correcto: false, incorrecto: false})
                    Object.assign(this.barcodePosicionDestino, {dato: "", mostrar: false, correcto: false, incorrecto: false})
                    this.cantidadArticulos.mostrar=false
                    this.listaArticulosOK=[]
                    this.listaBarcodesArticulos=[]
                    this.enfocarBarcodePosicionOrigen()

                })
                .catch(error => {
                    this.mostrarError(error)
                    console.log("Error", error)
                })
            } else {
                //reposicionamiento comun
                const payload={
                    vaciarPosicionOrigen: this.posicionQuedaVacia, 
                    articulos,
                    posicionOrigen: this.barcodePosicionOrigen.dato,
                    posicionDestino: this.barcodePosicionDestino.dato,
                    usuario: this.userName,
                }
                //cargar al payload las cantidades
                productos.registrarReposicionamiento(payload)
                .then(respuesta => {
                    for(const resultados of respuesta){
                        if(resultados.unProducto != undefined){
                            if(resultados.result.status == "OK"){
                                this.listaCodigosLeidos.unshift({Barcode: resultados.unProducto.Barcode, Correcto: true, Descripcion: resultados.unProducto.Nombre})
                            }else{
                                this.listaCodigosLeidos.unshift({Barcode: resultados.unProducto.Barcode, Correcto: false, Descripcion: resultados.result.status})
                            }
                        }
                    }
                    const mensajeExito={titulo: "El proceso a finalizado", mensaje: " "}
                    store.dispatch("alertDialog/mostrar", mensajeExito)
                    Object.assign(this.barcodePosicionOrigen, {dato: "", correcto: false, incorrecto: false})
                    Object.assign(this.barcodePosicionDestino, {dato: "", mostrar: false, correcto: false, incorrecto: false})
                    this.cantidadArticulos.mostrar=false
                    this.listaArticulosOK=[]
                    this.listaBarcodesArticulos=[]
                    this.enfocarBarcodePosicionOrigen()

                })
                .catch(error => {
                    this.mostrarError(error)
                    console.log("Error", error)
                })
            }
        },
        async procesarBarcodeArticulo() {
            this.articuloActual = null;
            try {
                // Fetch product details to get its dimensions
                this.articuloActual = await productos.getByBarcodeAndEmpresa(this.barcodeArticulo, this.idEmpresa);
            } catch (error) {
                this.mostrarError("Artículo no encontrado.");
                this.barcodeArticulo = "";
                return;
            }

            if(this.tieneLOTE == true){
                this.cantidadArticulos.mostrar=false
                this.cantidad = 1
                let lote = this.barcodeArticulo
                await productos.getLote(this.idEmpresa,lote)
                .then(response => {
                    if(response == 0){
                        const mensajeError = {titulo: "Error", mensaje:"El lote no existe "}
                        store.dispatch("alertDialog/mostrar", mensajeError)
                        this.barcodeArticulo=""
                        Object.assign(this.barcodePosicionDestino, {dato: "", mostrar: false, correcto: false, incorrecto: false})
                    }else{
                        let barcode
                        for(const unLote of response){
                            barcode=unLote.SerialNumber.split("-")
                        }
                        let contadorBucle=1
                        for(const barcodeProducto of barcode)
                        {
                            productos.getByBarcodeAndEmpresa(barcodeProducto.trim(), this.idEmpresa)
                            .then(respuesta => {
                                this.articuloActual = respuesta; // Store for validation
                                for(const articulos of respuesta.Posiciones){
                                    if(articulos.Nombre == this.barcodePosicionOrigen.dato){
                                        contadorBucle =0
                                        productos.getPosicionesPorProductoAndLote(respuesta.Id,articulos.Id, lote)
                                        .then(responsePos => {
                                            if(!responsePos){
                                                const mensajeError = {titulo: "Error", mensaje:"El lote no pertenece a la posicion de origen"}
                                                store.dispatch("alertDialog/mostrar", mensajeError)
                                            }else{
                                                respuesta.Embarque = response[0].Embarque
                                                respuesta.cantidad = responsePos.total
                                                respuesta.lote = lote
                                                this.listaBarcodesArticulos.push([barcodeProducto.trim(),  responsePos.total, lote])
                                                this.listaArticulosOK.push(respuesta)
                                            }
                                        })
                                        .catch(error => {
                                            this.mostrarError(error)
                                            console.log("Error", error)
                                        })
                                    }else{
                                        if(contadorBucle == respuesta.Posiciones.length){
                                            contadorBucle == 0
                                            const mensajeError = {titulo: "Error", mensaje:"El lote no pertenece a la posicion de origen"}
                                            store.dispatch("alertDialog/mostrar", mensajeError)
                                            this.barcodeArticulo=""
                                            if(this.listaBarcodesArticulos.length() < 1){
                                                this.barcodePosicionDestino.mostrar=false
                                            }
                                        }else{
                                            contadorBucle++
                                        }
                                        
                                    }
                                }
                                this.barcodeArticulo=""
                            })
                            .catch(error => {
                                //todosExisten=false
                                console.log(error)
                                this.listaCodigosFallidos.push(barcodeProducto)
                            })
                        }
                        this.barcodePosicionDestino.mostrar=true
                        this.enfocarBarcodePosicionDestino() 
                    }
                })
                .catch(error => {
                    console.log(error)
                })
                
            }else if(this.manejaStockUnitario==true){
                this.cantidad = 1
                this.procesarCantidad(); // Go to validation
            }else{
                this.cantidadArticulos.mostrar=true
                this.enfocarCantidadArticulo()
            }
        },
        procesarCantidad(){
            if(!this.barcodeArticulo || !this.cantidad || this.cantidad<1){
                const mensajeError = {titulo: "Error", mensaje:"Los campos de barcode y posicion no pueden estar vacios"}
                store.dispatch("alertDialog/mostrar", mensajeError)
                return;
            }
            if (this.capacityErrorMessages.length > 0) {
                this.mostrarError("La cantidad excede la capacidad de la posición de destino.");
                return;
            }

            this.listaBarcodesArticulos.push([this.barcodeArticulo,  this.cantidad])
            this.barcodeArticulo=""
            this.cantidad=""
            this.articuloActual = null; // Reset for next item
            this.barcodePosicionDestino.mostrar=true
            this.enfocarBarcodePosicionDestino()
        },
        eliminarLectura(item, index) {
            if(this.tieneLOTE== true){
                this.listaArticulosOK = this.listaArticulosOK.filter(element => element.lote !== item[2])
                this.listaBarcodesArticulos = this.listaBarcodesArticulos.filter(element => element.every(lote => lote !== item[2]))
            }else{
                this.listaBarcodesArticulos.splice(index, 1)
            }
            if(this.listaBarcodesArticulos.length==0){
                Object.assign(this.barcodePosicionDestino, {dato: "", mostrar: false, correcto: false, incorrecto: false})
            }
        },
        eligioEmpresa() {
            this.manejaStockUnitario=false
            this.tieneLOTE=false
            this.listaBarcodesArticulos=[]
            this.listaArticulosOK=[]
            this.listaCodigosLeidos=[]
            this.listaCodigosFallidos=[]
            this.posicionDestinoOcupacion = null;
            empresas.getOneById(this.idEmpresa)
            .then(respuesta => {
                if(respuesta.LOTE == true){
                        this.tieneLOTE = true   
                    } else {
                        this.tieneLOTE = false        
                    }
                    if(respuesta.StockUnitario){
                        this.manejaStockUnitario=true
                    }else{
                        this.manejaStockUnitario=false
                    }
                    this.barcodeArticulo=""
                    Object.assign(this.barcodePosicionOrigen, {dato: "", correcto: false, incorrecto: false})
                    Object.assign(this.barcodePosicionDestino, {dato: "", mostrar: false, correcto: false, incorrecto: false})
                })
                .catch(error => {
                    this.mostrarSnackbar(error)
                    console.log("Error", error)
                })
            this.barcodePosicionDestino.mostrar = false
            this.barcodePosicionOrigen.mostrar=true
            this.enfocarBarcodePosicionOrigen()
        },
        procesarBarcodePosicionOrigen() {
            this.posicionDestinoOcupacion = null;
            posiciones.getByNombre(this.barcodePosicionOrigen.dato)
                .then(respuesta => {
                    this.posicionOrigen=respuesta
                    this.barcodePosicionOrigen.correcto=true
                    this.barcodePosicionOrigen.incorrecto=false
                    this.listaBarcodesArticulos=[]
                    this.listaArticulosOK=[]
                    this.barcodeArticulo=""
                    this.cantidad=""
                    this.cantidadArticulos.mostrar=false
                    Object.assign(this.barcodePosicionDestino, {dato: "", mostrar: false, correcto: false, incorrecto: false})
                    this.enfocarBarcodeArticulo()             
                })
                .catch(puteada => {
                    this.barcodePosicionOrigen.incorrecto=true
                    this.barcodePosicionOrigen.correcto=false
                    this.enfocarBarcodePosicionOrigen()
                })
        },
        async procesarBarcodePosicionDestino() {
            this.posicionDestinoOcupacion = null;
            if (this.barcodePosicionOrigen.dato === this.barcodePosicionDestino.dato) {
                this.mostrarError("La posición de destino no puede ser igual a la de origen")
                this.barcodePosicionDestino.incorrecto=true
                this.barcodePosicionDestino.correcto=false
                this.enfocarBarcodePosicionDestino()
                return;
            }

            try {
                const respuesta = await posiciones.getByNombre(this.barcodePosicionDestino.dato);
                this.posicionDestino = respuesta;
                this.barcodePosicionDestino.correcto = true;
                this.barcodePosicionDestino.incorrecto = false;
                this.enfocarBarcodeArticulo();

                // Fetch occupancy details
                this.posicionDestinoOcupacion = await posicionesV3.getOcupacionDetalle(respuesta.Id, this.idEmpresa);

            } catch (error) {
                this.mostrarError("Posición de destino inexistente o sin datos de capacidad.");
                this.barcodePosicionDestino.incorrecto = true;
                this.barcodePosicionDestino.correcto = false;
                this.enfocarBarcodePosicionDestino();
            }
        },
        enfocarBarcodePosicionOrigen() {
            this.$nextTick(() => {
                this.$refs.barcodePosicionOrigen.focus()
            })
        },
        enfocarBarcodePosicionDestino() {
            this.$nextTick(() => {
                this.$refs.barcodePosicionDestino.focus()
            })
        },
        enfocarBarcodeArticulo() {
            this.$nextTick(() => {
                this.$refs.barcodeArticulo.focus()
            })
        },
        enfocarCantidadArticulo() {
            this.$nextTick(() => {
                this.$refs.cantidad.focus()
            })
        },
        mostrarError(mensaje) {
            store.dispatch("alertDialog/mostrar", {titulo: "Se ha producido un error", mensaje})
        },

        fileOnChange(archivoLeido) {
            //"manejaStockUnitario || tieneLOTE"
            if(this.manejaStockUnitario || this.tieneLOTE){
            
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
            }else{
                if (archivoLeido!=null) {
                    const lector=new FileReader()
                    lector.onload = ev => {
                    const datosCrudos = ev.target.result;
                    const planillaCruda = read(datosCrudos, {type: "binary", cellDates: true, cellNF: true, cellText:true})
                    const nombreHoja1=planillaCruda.SheetNames[0]
                    const datosPlanilla=utils.sheet_to_json(planillaCruda.Sheets[nombreHoja1])
                    this.verificarColumnasExcelMasivo(datosPlanilla)
                    }

                    lector.readAsBinaryString(archivoLeido)            
                }
            }
        },

        verificarColumnasExcel(planilla) {

        let columnasObligatorias= (this.tieneLOTE ? ['Posicion', 'BoxNumber'] : ['Posicion', 'Barcode Producto'])
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
                    mensaje += ";Fila: "+filaActual+" - Columna: "+unaColumnaObligatoria
                }
            }
            })
        })
            if (tituloMostrado) {
                store.dispatch("alertDialog/mostrar", {titulo: "No se puede procesar", mensaje})
                //this.mostrarMensaje("No se puede procesar", mensaje)
            } else {
                this.procesarPlanilla(planilla)
            }
        },

        verificarColumnasExcelMasivo(planilla) {

            let columnasObligatorias= (['Posicion Origen', 'Barcode Producto', 'Posicion Destino', 'Cantidad'])
            let revision = planilla[0]
            if(!Object.keys(revision).includes('Code Empresa')){
                //si no incluye code empresa en los campos usa estas
                columnasObligatorias= (['Posicion Origen', 'Barcode Producto', 'Posicion Destino', 'Cantidad'])
                this.tipoCodigo = 'Barcode Producto'
            }else{
                //si incluye code empresa usa estas
                columnasObligatorias = (['Posicion Origen', 'Code Empresa', 'Posicion Destino', 'Cantidad'])
                this.tipoCodigo= 'Code Empresa'
            }
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
                        mensaje += ";Fila: "+filaActual+" - Columna: "+unaColumnaObligatoria
                    }
                }
                })
            })
            if (tituloMostrado) {
                store.dispatch("alertDialog/mostrar", {titulo: "No se puede procesar", mensaje})
                //this.mostrarMensaje("No se puede procesar", mensaje)
            } else {
                this.procesarPlanilla(planilla)
            }
        },

        async  descargarExcelEjemplo(){
            const workbook=new excel.Workbook()
            const worksheet=workbook.addWorksheet("Pedidos")
            let cabeceras = []
            let renglon=1
            ['']
            worksheet.views = [{state: 'frozen', ySplit: 1}]
            console.log(this.tienePART)
            if(this.tienePART){
                cabeceras=[
                {header: 'Fecha', width: 11}, 
                {header: 'Comprobante', width: 20}, 
                {header: 'Cliente', width: 30}, 
                {header: 'Partida', width: 20}, 
                {header: 'Barcode Producto', width: 20}, 
                {header: 'Cantidad', width: 15}, 
                {header: 'Importe', width: 30}, 
                {header: 'Depósito origen', width: 30}, 
                {header: 'Empresa', width: 10}, 
                {header: 'Codigo Postal', width: 15}, 
                {header: 'Domicilio', width: 30}, 
                {header: 'Nota del cliente', width: 20}, 
                {header: 'Método de envío', width: 20}, 
                {header: 'Orden de retiro OCA', width: 20}, 
                {header: 'Observaciones', width: 30}, 
                {header: 'Email Destinatario', width: 20}, 
                {header: 'PreOrden', width: 10}, 
            ]
            }else{
                cabeceras=[
                    {header: 'Posicion Origen', width: 25}, 
                    {header: 'Barcode Producto', width: 25}, 
                    {header: 'Posicion Destino', width: 30}, 
                    {header: 'Cantidad', width: 20}, 
                ]
            }

            worksheet.columns=[...cabeceras]
            renglon++
                                

            worksheet.eachRow ( (row, rowNumber) => {
            row.eachCell ( (cell, colNumber) => {
                if (rowNumber==1) {
                    cell.font={size: 12, bold: true}
                } else {
                    if (rowNumber==renglon) {
                        cell.font={size: 12, bold: false}
                    } else {
                        cell.font={size: 10}
                    }
                }
                })
            } )

            const buf=await workbook.xlsx.writeBuffer()
            saveAs(new Blob([buf]), `Creacion_Ordenes_gestion.xlsx`)
        },

        async procesarPlanilla(planilla) {
            
            for (const unaFila of planilla) {
                
                if(this.tieneLOTE){
                    await posiciones.getByNombre(unaFila.Posicion)
                    .then(respuesta => {
                        unaFila.idPosicion = respuesta.Id
                        //reposicionamiento de lotess por excel
                        productos.updatePosicionByLoteAndIdPosicion(unaFila.BoxNumber, unaFila.idPosicion, this.userName)
                        .then(response => {
                            response.forEach(posicion => {
                                if(posicion.status){
                                    this.listaCodigosFallidos.unshift({Lote: posicion.lote, Correcto: false,Posicion: posicion.posicion, IdProducto: posicion.IdProducto, Mensaje: posicion.mensaje})
                                }else{
                                    this.listaCodigosLeidos.unshift({Lote: posicion.Lote, Correcto: true, IdProducto: posicion.IdProducto, Unidades: posicion.Unidades, Posicion: unaFila.Posicion})
                                }
                            })
                        })
                        .catch(puteada => {
                            console.error(puteada)
                        })
                    })
                    .catch(puteada => {
                        console.error(puteada)
                        this.listaCodigosFallidos.unshift({Lote: unaFila['BoxNumber'], Correcto: false, IdProducto: "ERROR", Unidades: "ERROR", Posicion: puteada})
                    })
                } else if(this.manejaStockUnitario){
                    try {
                        const posicion=await posiciones.getByNombre(unaFila.Posicion)
                        try {
                            const articulo=await productos.getByBarcodeAndEmpresa(unaFila['Barcode Producto'], this.idEmpresa)
                            try {
                                 const response=await productos.registraReposicionamientoExcel(this.idEmpresa,articulo.Id, posicion.Id, 1,this.userName)
                                 this.listaCodigosLeidos.unshift({Barcode: unaFila['Barcode Producto'], Correcto: true, Descripcion: articulo.Nombre})
    
                            } catch (error) {
                                this.listaCodigosFallidos.unshift({Barcode: unaFila['Barcode Producto'], Correcto: false, Descripcion: error})
                            }
                        } catch (error) {
                            console.log("Error en articulo", error)
                            this.listaCodigosFallidos.unshift({Barcode: unaFila['Barcode Producto'], Correcto: false, Descripcion: error})
                        }
                    } catch (error) {
                        console.log("Error en posicion", error)
                        this.listaCodigosFallidos.unshift({Barcode: unaFila['Barcode Producto'], Correcto: false, Descripcion: error})
                    }
                }else{
                    try {
                        let articulo = ''
                        try {
                            
                            if(this.tipoCodigo == "Code Empresa"){
                                //metodo que busca usando el code empresa
                                articulo = await productos.getByCodeEmpresaAndEmpresa(unaFila['Code Empresa'], this.idEmpresa)
                            }else{
                                //metodo que busca usando el barcode
                                articulo=await productos.getByBarcodeAndEmpresa(unaFila['Barcode Producto'], this.idEmpresa)
                            }

                            try {
                                 const response=await productos.registraReposicionamientoExcelCantidad(this.idEmpresa,articulo.Id, unaFila['Posicion Origen'], unaFila['Posicion Destino'], unaFila.Cantidad,unaFila[this.tipoCodigo],this.userName)
                                 this.listaCodigosLeidos.unshift({Barcode: unaFila[this.tipoCodigo], Correcto: true, Descripcion: articulo.Nombre})
    
                            } catch (error) {
                                console.log(error)
                                this.listaCodigosFallidos.unshift({Barcode: unaFila[this.tipoCodigo], Correcto: false, Descripcion: error})
                            }
                        } catch (error) {
                            console.log("Error en articulo", error)
                            this.listaCodigosFallidos.unshift({Barcode: unaFila[this.tipoCodigo], Correcto: false, Descripcion: error})
                        }
                    } catch (error) {
                        console.log("Error en posicion", error)
                        this.listaCodigosFallidos.unshift({Barcode: unaFila[this.tipoCodigo], Correcto: false, Descripcion: error})
                    }
                }
            }
        },
    },

    mounted() {
        store.dispatch('actualizarTituloPrincipal', 'Reposicionamiento de stock')
        empresas.getAll(true)
            .then(respuesta => {
                this.listaEmpresas=respuesta
            })
            .catch(puteada => {
                console.error(puteada)
            })
            this.userName =  store.state.usuarios.usuarioActual.Nombre
    },
}
</script>