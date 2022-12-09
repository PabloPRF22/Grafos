<template>
     <v-navigation-drawer v-model = "show" absolute stateless left :width="400">
    <v-toolbar color="primaryLight" dark dense :height = "60">
      <v-toolbar-title>Informacion del nodo</v-toolbar-title>
      <v-btn>Hola</v-btn>
    </v-toolbar>
      <v-divider></v-divider>
      <v-img src = "https://www.lavanguardia.com/files/image_449_220/uploads/2019/05/17/5fa539e91e129.jpeg"   max-height="300" max-width="400"></v-img>
  </v-navigation-drawer>
</template>
  
  <script>
  export default {
    name: "network",
    props: {
      show: Boolean,
    },
  
    data() {
      return {
   
      };
    },
    computed: {},
    watch: {

    },
  
    created() {},
    mounted() {},
    methods: {},
  };
  </script>
  
  <style scoped>
  .setting-box-card {
    width: 300px;
    height: auto;
    position: absolute;
  }
  </style>
  