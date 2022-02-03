<template>
  <v-data-table
    :headers="headers"
    :items="notdesserts"
    sort-by="calories"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Produtos</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="editDialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }"> </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.title"
                      label="Title"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.type"
                      label="Type"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.description"
                      label="Description"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.rating"
                      label="Rating"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.price"
                      label="Price"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
              <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <div class="d-flex">
              <client-only>
                <v-file-input
                  v-model="file"
                  counter
                  show-size
                  truncate-length="15"
                  small-chips
                  label="Insert File Here"
                ></v-file-input>
              </client-only>

              <div class="d-flex justify-end mt-3 ml-3">
                <v-btn color="primary" @click="submitFile" elevation="2"
                  >Importar</v-btn
                >
              </div>
            </div>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete"
                >Cancel</v-btn
              >
              <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template v-slot:no-data> </template>
  </v-data-table>
</template>
<script>
import axios from "axios";
const api = axios.create({
  baseURL: "http://localhost:8001/api/produtos/",
});
import Swal from "sweetalert2";
export default {
  data: () => ({
    file: null,
    dialog: false,
    editDialog: false,
    dialogDelete: false,
    headers: [
      { text: "Title", value: "title" },
      { text: "Type", value: "type" },
      { text: "Description", value: "description" },
      { text: "Rating", value: "rating" },
      { text: "Price", value: "price" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    desserts: [],
    notdesserts: [],
    editedIndex: -1,
    editedItem: [],
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    editDialog(val) {
      val || this.close();
    },

    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.produtos();
  },

  methods: {
    async produtos() {
      api
        .get()
        .then((response) => {
          const { data } = response;
          this.notdesserts = data;
        })
        .catch((err) => {
          console.error("ops! ocorreu um erro" + err);
        });
    },
    editItem(item) {
      this.editedIndex = this.notdesserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.editDialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.notdesserts.indexOf(item);
      this.deleteItemConfirm(item);
    },

    deleteItemConfirm(item) {
      let itemId = item.id;
      Swal.fire({
        title: `Want to remove ${item.title} ?`,
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        reverseButtons: true,
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!",
      }).then((result) => {
        if (result.isConfirmed) {
          api
            .delete(`${itemId}`, {
              headers: {
                "Content-Type": "application/json",
              },
            })
            .then(() => {
              Swal.fire("Deleted!", "Your file has been deleted.", "success");
              this.produtos();
            });
        }
      });
    },
    close() {
      this.dialog = false;
    },
    closeEditDialog() {
      this.editDialog = false;
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
      await api.put(`${this.editedItem.id}`, this.editedItem);
      this.produtos();
      this.closeEditDialog();
    },
    submitFile() {
      let self = this;
      let formData = new FormData();
      formData.append("json", this.file);
      console.log(">> formData >> ", formData);

      // You should have a server side REST API

      api
        .post("", formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        })
        .then(() => {
          this.successAlert()
          this.close();
        })
        .catch(function (e) {
          console.log(e);
        });
    },
    successAlert() {
      Swal.fire({
        toast: true,
        icon: "success",
        title: "Registered successfully",
        position: "top-end",
        showConfirmButton: false,
        timer: 3000,
        timerProgressBar: true,
        didOpen: (toast) => {
          toast.addEventListener("mouseenter", Swal.stopTimer);
          toast.addEventListener("mouseleave", Swal.resumeTimer);
        },
      });
    },

  },
};
</script>
