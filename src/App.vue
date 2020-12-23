<template>
  <div id="app">
    <h1>Dataset Cropper</h1>

    <ImageCropper :api="api" />
  </div>
</template>

<script>
import ImageCropper from "./components/ImageCropper.vue";

export default {
  name: "App",
  components: {
    ImageCropper
  },
  data(){
    return {
      api: ""
    }
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  >h1{
    margin: 0.25em;
  }
}
.apiLabel {
  font-weight: bold;
}
body {
  background-color: rgb(248,248,255);
}
.moveable-line {
     display: none;
}
</style>
