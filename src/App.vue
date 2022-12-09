<template>
  <v-app>
    <v-btn
      fab
      dark
      small
      :style="{margin : '10px auto'}"
    
      @click="showAjustes=!showAjustes"
    >
      <v-icon dark>
        mdi-cog-outline
      </v-icon>
    </v-btn>
    <popup :show = "showNodeInfo"       @showNodeInfo = "showInfoNode"></popup>
    <ajustes 
      :showAjustes= "showAjustes"      
      :nodeSize="nodeSize"
      :linkWidth="linkWidth"
      :linkDistance="linkDistance"
      :bodyStrength="bodyStrength"
      @modifySettings = "modifySettings"

    ></ajustes>
    <network
      :nodeList="nodes"
      :linkList="links"
      :nodeSize="nodeSize"
      :linkWidth="linkWidth"
      :linkDistance="linkDistance"
      :bodyStrength="bodyStrength"
      @showNodeInfo = "showInfoNode"
    ></network>
  </v-app>
</template>

<script>
import Network from "./components/Network.vue";
import Ajustes from "./components/Ajustes.vue"
import Popup from "./components/Popup.vue"
import axios from "axios";

export default {
  name: "app",
  components: {
    Network,
    Ajustes,
    Popup
  },
  data() {
    return {
      bgcolor:"white",
      nodes: [],
      links: [],
      showAjustes: false,
      showNodeInfo: false,
      nodeSize: 14,
      linkWidth: 2,
      linkDistance: 50,
      bodyStrength: -150,
    };
  },
  created() {
    let url ="/example.json";
    axios
      .get(url)
      .then(response => {
        this.nodes = response.data.nodes;
        this.links = response.data.links;
      })
      .catch(err => console.log(err));
  },
  methods: {
    showInfoNode(visibilidad){
      this.showNodeInfo = visibilidad

    },
    modifySettings(id,value){
      switch (id) {
        case 0:
          this.nodeSize = value
          break;
        case 1:
          this.linkWidth = value
          break;
        case 2:
          this.linkDistance = value
          break;
        case 3:
          this.bodyStrength = value
          break;
        default:
          break;
      }

  }
  }
};
</script>

<style>
body {
  margin: 0;
}
.setting-button {

}



</style>
