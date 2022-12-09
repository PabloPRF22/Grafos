<template>
  <div id="network" :style="{ width: '100%', height: '100%' }">
    <div class="linkText" :style="linkTextPosition" v-show="linkTextVisible" v-text="linkTextContent"></div>
    <svg width="100vw" height="100%" :style="{ 'background-color': theme.bgcolor }" @click="clickEle"
      @mouseover.prevent="svgMouseover" @mouseout="svgMouseout">
      <g id="container">
        <g>
          <g v-for="link in links" :key="link.index">
            <line :class="`${link[linkTypeKey]} ${link.selected} link element`" :stroke="theme.linkStroke"
              :stroke-width="linkWidth"></line>
            <text v-if="showLinkText" dx="0" dy="0" class="link-text" :fill="theme.textFill"
              :font-size="linkTextFrontSize">{{ link[linkTextKey] }}</text>
          </g>
        </g>

        <g id="node-group">
          <g v-for="node in nodes" :key="node.index">
            <circle :fill="nodeColor(node[nodeTypeKey])" :stroke-width="highlightNodes.indexOf(node.id) == -1 ? 3 : 10"
              :stroke="highlightNodes.indexOf(node.id) == -1 ? theme.nodeStroke : 'gold'"
              :class="`${node.show ? 'selected' : ''} node element`" :r="nodeSize"></circle>
            <text v-show="node.show || showAllText" class="node-text" :fill="theme.textFill"
              :font-size="nodeTextFontSize" x="50%" y="50%" text-anchor="middle">{{ node[nodeTextKey] }}</text>
          </g>
          <g></g>
        </g>
      </g>
    </svg>
  </div>
  <!-- </div> -->
</template>

<script>
import * as d3 from "d3";
import { select, svg } from "d3";


export default {
  name: "network",
  props: {
    nodeList: Array,
    linkList: Array,
    // node
    nodeSize: {
      type: Number,
      default: 10
    },
    nodeTextKey: {
      type: String,
      default: "id"
    },
    nodeTypeKey: {
      type: String,
      default: "group"
    },
    nodeTextFontSize: {
      type: Number,
      default: 14
    },
    // link
    linkWidth: {
      type: Number,
      default: 2
    },
    showLinkText: {
      type: Boolean,
      default: false
    },
    linkTextKey: {
      type: String,
      default: "value"
    },
    linkTypeKey: {
      type: String,
      default: "type"
    },
    linkTextFrontSize: {
      type: Number,
      default: 10
    },
    linkDistance: {
      type: Number,
      default: 50
    },
    // svg
    svgSize: {
      type: Object,
      default: () => {
        return {
          width: window.innerWidth,
          height: window.innerHeight
        };
      }
    },
    bodyStrength: {
      type: Number,
      default: -150
    },
    // others
    highlightNodes: {
      type: Array,
      default: () => {
        return [];
      }
    }
  },
  data() {
    return {
      selection: {
        links: [],
        nodes: []
      },
      pinnedNode: null,
      pinned: [],
      force: null,
      showAllText: true,
      zoom: d3.zoom(), //INICIALIZA EL COMPORTAMIENTO DEL ZOOM
      nodeColor: d3.scaleOrdinal(d3.schemeCategory10),
      linkTextVisible: false,
      linkTextPosition: {
        top: 0,
        left: 0
      },
      linkTextContent: "",
      size: {
        height: 500,
        width: 500

      }

    };
  },
  computed: {
    nodes() {
      let nodes = this.nodeList.slice();
      let nodeIds = [];
      nodes = nodes.filter(node => {
        if (nodeIds.indexOf(node.id) === -1) {
          nodeIds.push(node.id);
          return true;
        } else {
          return false;
        }
      });
      return nodes;
    },
    links() {
      return this.linkList;
    },
    theme() {
      return {
        bgcolor: "#FFD500",
        nodeStroke: "white",
        linkStroke: "black",
        textFill: "black"
      };
    },
    center() {

      return {
        x: this.size.width / 2 + (this.size.width / 200),
        y: this.size.height / 2 + (this.size.height / 200)
      }
    }

  },
  watch: {
    bodyStrength: function () {
      this.initData();
      this.$nextTick(function () {
        this.initDragTickZoom();
      });
    },
    linkDistance: function () {
      this.initData();
      this.$nextTick(function () {
        this.initDragTickZoom();
      });
    },
    nodes: function () {
      this.initData();
      this.$nextTick(function () {
        this.initDragTickZoom();
      });
    }
  },

  created() {
    this.initData();
  },
  mounted() {
    this.onResize()
    this.initDragTickZoom();
    this.initCss()
    window.addEventListener('resize', this.onResize)
  },
  methods: {
    onResize() {
      this.size.width = this.$el.clientWidth
      this.size.height = this.$el.clientHeight
      this.$forceUpdate();



    },
    initCss() {
      d3.selectAll(".node")
        .attr("width", 10)

    },
    initData() {
      this.force = d3
        .forceSimulation(this.nodes)
        .force(
          "link",
          d3
            .forceLink(this.links)
            .id(d => d.id)
            .distance(this.linkDistance)
        )
        .force("charge", d3.forceManyBody().strength(this.bodyStrength)) //The strength of the attraction or repulsion
        .force(
          "center",
          d3.forceCenter(this.center.x, this.center.y)
        )
        
      this.force.nodes().forEach(function (node) {
        node.on("click", function () {
          console.log("eem")
        });
      })

    },
    initDragTickZoom() {
      //Añadimos la funcionalidad de arrastrar y soltar los nodos
      d3.selectAll(".node").call(this.drag(this.force));
      this.force.on("tick", () => {
        // ACTUALIZA LAS CORDENADAS DE LOS ENLANCES
        d3.selectAll(".link")
          .data(this.links)
          .attr("x1", d => d.source.x)
          .attr("y1", d => d.source.y)
          .attr("x2", d => d.target.x)
          .attr("y2", d => d.target.y);
        // ACTUALIZA LAS CORDENADAS DEL NODO
        d3.selectAll(".node")
          .data(this.nodes)
          .attr("cx", d => d.x)
          .attr("cy", d => d.y);
        // ACTUALIZA LAS CORDENADAS DEL TEXTO DEL NODO
        d3.selectAll(".node-text")
          .data(this.nodes)
          .attr("x", d => d.x)
          .attr("y", d => d.y);
        //ACTUALIZA LAS COORDENADAS DEL TEXTO DEL ENLANCE
        d3.selectAll(".link-text")
          .data(this.links)
          .attr("x", d => (d.source.x + d.target.x) / 2)
          .attr("y", d => (d.source.y + d.target.y) / 2);
      });

      // Inicializacion del zoom
      //scaleExtent set the allowed scale range
      this.zoom.scaleExtent([0.1, 3]).on("zoom", this.zoomed);

      d3.select("svg")
        .call(this.zoom)
        .on("dblclick.zoom", null);
    },
    clickLink(e) {
      //this.$emit("clickLink", e, e.target.__data__);
    },
    clickNode(e) {
      if (this.pinned.length === 0) {
        this.$emit("showNodeInfo", true)
        this.pinnedState(e);
      } else {
        this.$emit("showNodeInfo", false)
        d3.selectAll(".element").style("opacity", 0.2);
        this.pinned = [];
      }
      //this.$emit("clickNode", e, e.target.__data__);
    },
    clickEle(e) {
      if (e.target.tagName === "circle") {
        const node = e.target

        this.clickNode(e);


      } else if (e.target.tagName === "line") {
        this.clickLink(e);
      } else {
        if (this.pinnedNode !== null) {
          this.$emit("showNodeInfo", false)
          this.pinned = [];
          this.svgMouseout(this.pinnedNode)
          this.pinnedNode = null
        }
      }
    },
    selectNode(node) {
      console.log("entro")
      this.updateGraph(node)
    },
    updateGraph(selectNode) {
      d3.selectAll(".node").each(function (node) {
        if (node == selectNode) {
          console.log(node)

        }
      })

    },
    svgMouseover(e) {

      if (e.target.nodeName === "circle") {
        if (this.pinned.length === 0) {
          this.selectedState(e);
        }
        this.$forceUpdate();
      } else if (e.target.nodeName === "line") {
        this.linkTextPosition = {
          left: e.clientX + "px",
          top: e.clientY - 50 + "px"
        };
        this.linkTextContent = e.target.__data__[this.linkTextKey];
        this.linkTextVisible = true;
      }
    },
    svgMouseout(e) {

      this.linkTextVisible = false;
      if (e.target.nodeName === "circle") {
        if (this.pinned.length === 0) {
          this.noSelectedState(e);
        }
        this.$forceUpdate();
      }
    },
    selectedState(e) {

      e.target.__data__.show = true;
      this.showAllText = false
      e.target.classList.add("selected");
      this.selection.nodes.push(e.target.__data__);
      this.lightNeibor(e.target.__data__);
      d3.selectAll(".element").style("opacity", 0.2);
    },
    noSelectedState(e) {
      e.target.__data__.show = false;
      this.showAllText = true
      this.darkenNerbor();
      d3.selectAll(".element").style("opacity", 1);
    },
    pinnedState(e) {
      this.pinnedNode = e
      this.pinned.push(e.target.__data__.index);
      d3.selectAll(".element").style("opacity", 0.05);

    },
    lightNeibor(node) {

      this.links.forEach(link => {
        if (link.source.index === node.index) {

          link.selected = "selected";
          this.selection.links.push(link);
          this.selection.nodes.push(link.target);
          this.lightNode(link.target);
        }
        if (link.target.index === node.index) {
          link.selected = "selected";
          this.selection.links.push(link);
          this.selection.nodes.push(link.source);
          this.lightNode(link.source);
        }
      });
    },
    lightNode(selectedNode) {

      this.nodes.forEach(node => {
        if (node.index === selectedNode.index) {
          node.show = true;
        }
      });
    },
    darkenNerbor() {
      this.links.forEach(link => {
        this.selection.links.forEach(selectedLink => {
          if (selectedLink.id === link.id) {
            link.selected = "";
          }
        });
      });
      this.nodes.forEach(node => {
        this.selection.nodes.forEach(selectednode => {
          if (selectednode.id === node.id) {
            node.show = false;
          }
        });
      });
      this.selection.nodes = [];
      this.selection.links = [];
    },
    zoomed(event) {
      // Zoom: centrado en la posición del ratón
      d3.select("#container").attr(
        "transform",
        "translate(" +
        event.transform.x +
        "," +
        event.transform.y +
        ") scale(" +
        event.transform.k +
        ")"
      );
    },
    drag(simulation) {

      function dragstarted(event, d) {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;
      }

      function dragged(event, d) {
        d.fx = event.x;
        d.fy = event.y;
      }

      function dragended(event, d) {
        if (!event.active) simulation.alphaTarget(0);
        d.fx = null;
        d.fy = null;
      }

      return d3
        .drag()
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended);
    }
  }
};
</script>

<style scoped>
svg {
  margin: 0 auto;
}

.element {
  transition: opacity 0.5s ease;
}

.selected {
  opacity: 1 !important;
}

.node,
.link {
  cursor: pointer;
}

.linkText {
  position: absolute;
  z-index: 10;
  background-color: rgba(75, 75, 75, 0.596);
  border-radius: 10px;
  color: white;
  padding: 10px;
}
</style>

