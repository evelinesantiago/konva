<template>
  <div>
    <v-stage
      ref="stage"
      :config="stageSize"
      @mousedown="handleStageMouseDown"
    >
      <v-layer ref="layer">
        <!-- Elipses confirmadas -->
        <v-ellipse
          v-for="item in confirmedEllipses"
          :key="'confirmed-' + item.id"
          :config="{
            ...item,
            opacity: 0.6,
            stroke: '#2e7d32',
            strokeWidth: 3,
            fill: '#4caf50',
            draggable: false,
          }"
        />

        <!-- Elipse em edição -->
        <v-ellipse
          v-if="currentEllipse"
          :config="{
            ...currentEllipse,
            fill: '#2196f3',
            opacity: 0.5,
            draggable: true,
          }"
          @transformend="handleTransformEnd"
        />

        <!-- Transformer para a elipse atual -->
        <v-transformer
          v-if="currentEllipse"
          ref="transformer"
          :config="{
            rotateEnabled: true,
            keepRatio: false,
            enabledAnchors: ['top-left','top-right','bottom-left','bottom-right'],
          }"
        />
      </v-layer>
    </v-stage>

    <!-- Botão de confirmar -->
    <div
      v-if="currentEllipse"
      style="position: fixed; bottom: 20px; left: 20px;"
    >
      <button
        @click="confirmEllipse"
        style="padding: 8px 16px; background: #4caf50; color: #fff; border: none; border-radius: 6px;"
      >
        Confirmar
      </button>
    </div>

    <!-- Modal simples -->
    <div
      v-if="showModal"
      style="
        position: fixed;
        top: 0; left: 0;
        width: 100vw; height: 100vh;
        background: rgba(0,0,0,0.5);
        display: flex; align-items: center; justify-content: center;
      "
    >
      <div
        style="
          background: white;
          padding: 20px;
          border-radius: 8px;
          min-width: 300px;
          display: flex;
          flex-direction: column;
          gap: 12px;
        "
      >
        <h3>Descrição</h3>
        <textarea v-model="currentText" rows="4" style="width: 100%; resize: none;"></textarea>
        <div style="display:flex; justify-content: flex-end; gap: 10px;">
          <button @click="cancelModal" style="padding:6px 12px;">Cancelar</button>
          <button @click="saveModal" style="padding:6px 12px; background:#4caf50; color:white; border:none;">Salvar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
const width = window.innerWidth
const height = window.innerHeight

export default {
  data() {
    return {
      stageSize: { width, height },

      // apenas uma elipse em edição
      currentEllipse: null as any | null,
      confirmedEllipses: [] as any[],

      // modal
      showModal: false,
      currentText: '',
      nextId: 1,
    }
  },
  methods: {
    handleStageMouseDown(e: any) {
      // só cria nova se não houver elipse em edição
      if (this.currentEllipse) return

      const stage = e.target.getStage()
      if (e.target === stage) {
        const pos = stage.getPointerPosition()
        if (pos) {
          this.currentEllipse = {
            id: this.nextId++,
            name: `ellipse${this.nextId}`,
            x: pos.x,
            y: pos.y,
            radiusX: 60,
            radiusY: 60,
            draggable: true,
          }
          this.$nextTick(() => {
            const transformerNode = this.$refs.transformer?.getNode?.()
            const shape = stage.findOne(`.${this.currentEllipse.name}`)
            if (transformerNode && shape) transformerNode.nodes([shape])
          })
        }
      }
    },

    handleTransformEnd(e: any) {
      if (!this.currentEllipse) return
      const node = e.target
      this.currentEllipse.x = node.x()
      this.currentEllipse.y = node.y()
      this.currentEllipse.rotation = node.rotation()
      this.currentEllipse.radiusX = node.radiusX() * node.scaleX()
      this.currentEllipse.radiusY = node.radiusY() * node.scaleY()
      node.scaleX(1)
      node.scaleY(1)
    },

    confirmEllipse() {
      // abre modal
      this.showModal = true
      this.currentText = ''
    },

    cancelModal() {
      this.showModal = false
    },

    saveModal() {
      if (!this.currentEllipse) return
      // salva elipse como estática
      this.confirmedEllipses.push({
        ...this.currentEllipse,
        text: this.currentText,
      })
      // reseta o estado
      this.currentEllipse = null
      this.showModal = false
    },
  },
}
</script>
