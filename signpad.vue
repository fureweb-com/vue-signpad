<template>
  <div class="signpad">
    <div class="buttonArea">
      <button class="btn" @click="clear">Clear</button>
      <button class="btn" v-if="localOptions.functions.download" @click="downloadImage">Download as Image</button>
    </div>
    <canvas ref="canvasPad" 
      v-bind:style="{border: localOptions.border}"
      v-bind:width="localOptions.width"
      v-bind:height="localOptions.height"
      @touchstart="onStart" @touchend="onEnd" @touchmove="onMove" 
      @mousedown="onStart" @mouseup="onEnd" @mousemove="onMove">
      Are you using a browser that supports CANVAS?
    </canvas>
  </div>
</template>

<script>
export default {
  name: 'vue-signpad',
  props: ['options'], // {width, height, border, lineColor, lineWidth, functions:{download: boolean, ...}}
  data () {
    return {
      localOptions: {},
      isDrawing: false,
      fileName: 'signpad',
      canvasPad: {},
      canvasPadContext: {},
      currentPosition: { x: 0, y: 0 },
      previousPosition: { x: 0, y: 0 }
    }
  },
  created() {
    // Apply default values if need
    this.localOptions = Object.assign({width: '200px', height: '100px', border: '1px solid black', lineColor: '#000000', lineWidth: 2, functions: {download: false}}, this.options)
  },
  mounted() {
    this.canvasPad = this.$refs.canvasPad
    this.canvasPadContext = this.canvasPad.getContext('2d')
  },
  methods: {
    getPosition(canvasDom, event) {
      const rect = canvasDom.getBoundingClientRect()
      return {
        x: String(event.type).includes('mouse') ? event.clientX - rect.left : event.touches[0].clientX - rect.left,
        y: String(event.type).includes('mouse') ? event.clientY - rect.top : event.touches[0].clientY - rect.top
      }
    },
    draw() {
      if (this.isDrawing) {
        this.canvasPadContext.beginPath()
        this.canvasPadContext.moveTo(this.previousPosition.x, this.previousPosition.y)
        this.canvasPadContext.lineTo(this.currentPosition.x, this.currentPosition.y)
        this.canvasPadContext.strokeStyle = this.localOptions.lineColor
        this.canvasPadContext.lineWidth = this.localOptions.lineWidth
        this.canvasPadContext.stroke()
        this.canvasPadContext.closePath()
        this.previousPosition = this.currentPosition
        this.$emit('draw', this.canvasPad)
      }
    },
    onStart(e) {
      e.preventDefault()
      this.isDrawing = true
      this.previousPosition = this.currentPosition
      this.currentPosition = this.getPosition(this.canvasPad, e)
    },
    onEnd(e) {
      this.isDrawing = false
    },
    onMove(e) {
      if (!this.isDrawing) return

      this.previousPosition = this.currentPosition
      this.currentPosition = this.getPosition(this.canvasPad, e)

      this.draw()
    },
    clear() {
      this.canvasPad.width = this.canvasPad.width
      this.$emit('clear')
    },
    downloadImage() {
      // TODO: background color issue
      const anchor = document.createElement('a')
      anchor.href = this.canvasPad.toDataURL()
      anchor.download = this.fileName
      anchor.click()
    }
  }
}
</script>

<style>

</style>
