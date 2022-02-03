<template>
  <div class="d-flex">
    <client-only>
      <v-file-input
        v-model="file"
        counter
        show-size
        truncate-length="15"
        small-chips
      ></v-file-input>
    </client-only>

    <div class="d-flex justify-end mt-3 ml-3">
      <v-btn color="primary" @click="submitFile" elevation="2">Importar</v-btn>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";
export default {
  data: () => ({
    file: "",
    arrayProdutos: "",
  }),
  name: "IndexPage",
  methods: {
    submitFile() {
      let formData = new FormData();
      formData.append("json", this.file);
      console.log(">> formData >> ", formData);

      // You should have a server side REST API
      axios
        .post("http://localhost:8001/api/produtos", formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        })
        .then(function () {
          this.close()
        })
        .catch(function () {
          console.log("FAILURE!!");
        });
    },

  },
};
</script>
