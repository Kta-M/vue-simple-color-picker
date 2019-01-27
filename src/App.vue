<template>
  <div id="app">
    <div style="margin-left: 200px;">
      <ColorPicker pickerPosition='right_bottom' :rgb.sync="rgb1" :alpha.sync="alpha1"/>
    </div>
  </div>
</template>

<script>
import ColorPicker from './components/ColorPicker.vue'

export default {
  name: 'app',
  components: {
    ColorPicker
  },
  data: function () {
    return {
      rgb1: { r: 100, g: 100, b: 100 },
      alpha1: 1
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
