<template>
  <v-container>
    <v-data-table
      :headers="headers"
      :items="categories"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Gestión de Categorías</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Nueva Categoría
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.nombre"
                        label="Nombre de la categoría"
                        :rules="[rules.required]"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">Cancelar</v-btn>
                <v-btn color="blue darken-1" text @click="save">Guardar</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5">¿Estás seguro de que quieres eliminar esta categoría?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancelar</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">mdi-pencil</v-icon>
        <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import posicionV3 from '@/store/posicionesV3';

export default {
  name: 'CategoriasAdmin',
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      { text: 'ID', value: 'id' },
      { text: 'Nombre', value: 'nombre' },
      { text: 'Acciones', value: 'actions', sortable: false },
    ],
    categories: [],
    editedIndex: -1,
    editedItem: {
      id: 0,
      nombre: '',
    },
    defaultItem: {
      id: 0,
      nombre: '',
    },
    rules: {
        required: value => !!value || 'Este campo es requerido.',
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Nueva Categoría' : 'Editar Categoría';
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    async initialize() {
      try {
        this.categories = await posicionV3.categorias.getAll();
      } catch (error) {
        console.error("Error fetching categories:", error);
      }
    },

    editItem(item) {
      this.editedIndex = this.categories.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.categories.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    async deleteItemConfirm() {
      try {
        await posicionV3.categorias.delete(this.editedItem.id);
        this.categories.splice(this.editedIndex, 1);
      } catch (error) {
        console.error("Error deleting category:", error);
      }
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    async save() {
      if (this.editedItem.nombre) {
        try {
            if (this.editedIndex > -1) {
                await posicionV3.categorias.update(this.editedItem.id, this.editedItem);
                Object.assign(this.categories[this.editedIndex], this.editedItem);
            } else {
                const newCategory = await posicionV3.categorias.create(this.editedItem);
                this.categories.push(newCategory);
            }
        } catch(error) {
            console.error("Error saving category:", error);
        }
        this.close();
      }
    },
  },
};
</script>
