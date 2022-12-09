<template>
  <v-navigation-drawer v-model = "showAjustes" absolute stateless right :width="400">
    <v-toolbar color="primaryLight" dark dense :height = "60">
      <v-toolbar-title>Ajustes</v-toolbar-title>
    </v-toolbar>
      <v-divider></v-divider>
      <v-slider v-model="nodeSize" label="Tamaño nodo" max="50" min="1">
        <template v-slot:append>
          <v-text-field
            v-model="nodeSize"
            class="mt-0 pt-0"
            hide-details
            single-line
            type="number"
            style="width: 60px"
          ></v-text-field>
        </template>
      </v-slider>

      <v-slider v-model="linkWidth" label="Grosor enlace" max="20" min="1">
        <template v-slot:append>
          <v-text-field
            v-model="linkWidth"
            class="mt-0 pt-0"
            hide-details
            single-line
            type="number"
            style="width: 60px"
          ></v-text-field>
        </template>
      </v-slider>
      <v-slider
        v-model="linkDistance"
        label="Tamaño de los enlaces"
        max="300"
        step="50"
      >
        <template v-slot:append>
          <v-text-field
            v-model="linkDistance"
            class="mt-0 pt-0"
            hide-details
            single-line
            type="number"
            style="width: 60px"
          ></v-text-field>
        </template>
      </v-slider>

      <v-slider
        v-model="bodyStrength"
        label="Fuerza"
        min="-1000"
        max="0"
        step="50"
      >
        <template v-slot:append>
          <v-text-field
            v-model="bodyStrength"
            class="mt-0 pt-0"
            hide-details
            single-line
            type="number"
            style="width: 60px"
          ></v-text-field>
        </template>
      </v-slider>
  </v-navigation-drawer>
</template>

<script>
export default {
  name: "network",
  props: {
    showAjustes: Boolean,
  },

  data() {
    return {
      nodeSize: 14,
      linkWidth: 2,
      linkDistance: 50,
      bodyStrength: -150,
    };
  },
  computed: {},
  watch: {
    nodeSize(newValue) {
      this.$emit("modifySettings", 0, newValue);
    },
    bodyStrength(newValue) {
      this.$emit("modifySettings", 3, newValue);
    },
    linkDistance(newValue) {
      this.$emit("modifySettings", 2, newValue);
    },
    linkWidth(newValue) {
      this.$emit("modifySettings", 1, newValue);
    },
  },

  created() {},
  mounted() {},
  methods: {},
};
</script>

<style scoped>
.setting-box-card {
  position: fixed;
  z-index: 10;
  overflow: auto;
  top: 15px;
  right: 15px;
  width: 500px;
  max-height: 700px;
}
</style>
