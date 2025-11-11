<template>
  <v-stage
    ref="stage"
    :config="stageSize"
    @mousedown="handleStageMouseDown"
    @touchstart="handleStageMouseDown"
  >
    <v-layer ref="layer">
      <v-ellipse
        v-for="item in ellipses"
        :key="item.id"
        :config="{
          ...item,
          fill: getFillColor(item.status),
          opacity: 0.7, // transparência reduzida (mais opaco)
        }"
        @transformend="handleTransformEnd"
        @dblclick="toggleStatus(item)"
      />
      <v-transformer
        ref="transformer"
        :config="{
          rotateEnabled: true,
          keepRatio: keepProportion, // true = círculo, false = elipse livre
          enabledAnchors: [
            'top-left',
            'top-right',
            'bottom-left',
            'bottom-right',
          ],
          boundBoxFunc: (oldBox, newBox) => {
            if (newBox.width < 5 || newBox.height < 5) return oldBox;
            return newBox;
          },
        }"
      />
    </v-layer>
  </v-stage>

  <!-- Controles -->
  <div
    style="position: fixed; bottom: 20px; left: 20px; display: flex; gap: 12px"
  >
    <button
      @click="toggleProportion"
      style="
        padding: 8px 14px;
        background: #1e88e5;
        color: #fff;
        border: none;
        border-radius: 6px;
      "
    >
      {{ keepProportion ? "🔓 Modo Elipse Livre" : "🔒 Modo Círculo Travado" }}
    </button>
  </div>
</template>

<script lang="ts">
import Konva from "konva";
const width = window.innerWidth;
const height = window.innerHeight;

export default {
  data() {
    return {
      stageSize: { width, height },
      keepProportion: true, // inicia como círculo (proporção travada)
      ellipses: [
        {
          id: 1,
          name: "ellipse1",
          x: 200,
          y: 200,
          radiusX: 60,
          radiusY: 60,
          status: "pendente", // 'pendente' | 'andamento' | 'concluido'
          draggable: true,
        },
      ],
      selectedShapeName: "",
      _nextId: 2, // contador simples para novos círculos
    };
  },
  methods: {
    getFillColor(status) {
      // cores sólidas; a transparência é controlada por "opacity"
      switch (status) {
        case "pendente":
          return "#FFC107"; // amarelo
        case "andamento":
          return "#2196F3"; // azul
        case "concluido":
          return "#4CAF50"; // verde
        default:
          return "#9E9E9E"; // cinza
      }
    },

    toggleProportion() {
      this.keepProportion = !this.keepProportion;
    },

    toggleStatus(item) {
      const next = {
        pendente: "andamento",
        andamento: "concluido",
        concluido: "pendente",
      };
      item.status = next[item.status] || "pendente";
    },

    // Cria um novo círculo/elipse na posição clicada
    addCircleAt(pointer) {
      const id = this._nextId++;
      const name = `ellipse${id}`;
      const newEl = {
        id,
        name,
        x: pointer.x,
        y: pointer.y,
        radiusX: 50, // tamanho padrão
        radiusY: 50, // igual para começar como círculo
        status: "pendente",
        draggable: true,
      };
      this.ellipses.push(newEl);
      // seleciona imediatamente o novo
      this.selectedShapeName = name;
      this.$nextTick(this.updateTransformer);
    },

    handleTransformEnd(e) {
      const el = this.ellipses.find((r) => r.name === this.selectedShapeName);
      if (!el) return;

      const node = e.target;
      el.x = node.x();
      el.y = node.y();
      el.rotation = node.rotation();
      el.radiusX = node.radiusX() * node.scaleX();
      el.radiusY = node.radiusY() * node.scaleY();
      node.scaleX(1);
      node.scaleY(1);
    },

    handleStageMouseDown(e) {
      const stage = e.target.getStage();

      // Se clicou no "vazio" do Stage, cria um novo círculo no ponto
      if (e.target === stage) {
        const pos = stage.getPointerPosition();
        if (pos) this.addCircleAt(pos);
        return;
      }

      // Ignora cliques no transformer
      const clickedOnTransformer =
        e.target.getParent().className === "Transformer";
      if (clickedOnTransformer) return;

      // Seleciona a elipse clicada (se houver)
      const name = e.target.name();
      const el = this.ellipses.find((r) => r.name === name);
      this.selectedShapeName = el ? name : "";
      this.updateTransformer();
    },

    updateTransformer() {
      const transformerNode = this.$refs.transformer.getNode();
      const stage = transformerNode.getStage();
      const selectedNode = stage.findOne("." + this.selectedShapeName);
      transformerNode.nodes(selectedNode ? [selectedNode] : []);
    },
  },
};
</script>
