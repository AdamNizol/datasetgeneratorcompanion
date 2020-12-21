<template>
  <div class="cropperContainer">

    <div class="controlsPanel">
      <button @click="newImg" class="newImgBtn">New Image</button>

      <div>
        <label class="apiLabel" for="api">API:</label>
        <input name="api" type="text" v-model="api" />
      </div>

      <button class="sendBtn">Send Selection</button>
    </div>

    <div class="instructionsContainer">
      <h4>{{ instructionSteps[instructionStep] }}</h4>
    </div>

    <div class="imageview">
      <img :src="imgData"  />
    </div>

  </div>
</template>

<script>
export default {
  name: "ImageCropper",
  props: {
    //st: String
  },
  data(){
    return {
      api: "https://dummyimage.com/wsvga",
      imgData: {},
      instructionSteps: [
        "Enter your API url",
        "Get a new image",
        "Mark smallest area containing your object",
        "Mark largest area containing your object",
        "Send your selection if it looks right"
      ],
      instructionStep: 0,
    }
  },
  methods: {
    newImg(){
      this.axios.get(this.api, { responseType: 'arraybuffer' }).then((response) => {
        let foo = Buffer.from(response.data, 'binary').toString('base64')
        this.imgData = "data:image/png;base64,"+foo;
      })
    },

  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.cropperContainer {
  .imageview{
    width: 600px;
    height: 600px;
    max-width: 95vw;
    border: 2px solid rgb(120,120,120);
    overflow: scroll;
  }
  .controlsPanel{
    background-color: rgb(220,220,230);
    border-radius: 5% 5% 0 0;
    border: 2px solid rgb(120,120,120);
    border-bottom: none;
    display: flex;
    justify-content: space-between;
    padding: 0.15em;
  }
  .instructionsContainer{
    background-color: rgb(220,220,230);
    border: 2px solid rgb(120,120,120);
    border-bottom: none;
    border-top: 1px solid rgb(180,180,185);
    padding: 0.15em;
    > h4{
      margin: 0;
      color: rgb(60,60,65);
    }
  }
}
</style>
