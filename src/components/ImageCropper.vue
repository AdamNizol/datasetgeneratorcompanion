<template>
  <div class="cropperContainer">

    <div class="controlsPanel">
      <div class="cpBtnLeft">
        <button @click="newImg" class="newImgBtn cpBtn">New Image</button>
      </div>

      <div>
        <label class="apiLabel" for="api">API:</label>
        <input name="api" type="text" v-model="api"  @change="changedAPI"/>
      </div>

      <div class="cpBtnRight">
        <button v-show="instructionStep>1" @click="sendSelection" class="sendBtn">Send Selection</button>
      </div>
    </div>

    <div class="instructionsContainer">
      <h4>{{ instructionSteps[instructionStep] }}</h4>
      <button class="nextBtn" v-if="instructionStep==2" @click="nextBtnClicked">Next</button>
    </div>

    <div class="imageview">
      <div class="imgContainer">
        <img ref="imgRef" :src="imgData" v-show="Object.keys(imgData).length > 0" :style="'transform: scale('+zoom+') translate('+offsetX+'px, '+offsetY+'px);'" />
      </div>

      <vue-draggable-resizable :key="regionKey" :w="defaultCropping.width" :h="defaultCropping.height" :resizable="true" @dragging="onDrag" @resizing="onResize" :parent="true" class="regionBox" v-if="instructionStep>1"></vue-draggable-resizable>

      <div class="scaleControls" v-if="instructionStep>1">
        <div class="scaleControlsRow">
          <button @click="zoom = Math.round( Math.max(0.10, zoom-0.2)*10 )/10">-</button>
          <input type="text" :value="zoom" readonly />
          <button @click="zoom = Math.round( Math.min(4, zoom+0.1)*10 )/10">+</button>
        </div>

        <div class="moveContainer">
          <div class="moveContainerColumn">
            <button @click="offsetX -= Math.round(25*(1/zoom))">⬅</button>
          </div>
          <div class="moveContainerColumn">
            <button @click="offsetY -= Math.round(25*(1/zoom))">⬆</button>
            <button @click="offsetY += Math.round(25*(1/zoom))">⬇</button>
          </div>
          <div class="moveContainerColumn">
            <button @click="offsetX += Math.round(25*(1/zoom))">➡</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable'
import 'vue-draggable-resizable/dist/VueDraggableResizable.css'

export default {
  name: "ImageCropper",
  components: {
    VueDraggableResizable
  },
  data(){
    let dCrop = {
      'x': 0,
      'y': 0,
      'width': 256,
      'height': 256
    }
    return {
      api: "http://localhost:5000/",
      imgData: {},
      imgUrl: "",
      instructionSteps: [
        "Enter your API url",
        "Get a new image",
        "Mark smallest area containing your object",
        "Send your selection if it looks right"
      ],
      instructionStep: 0,
      zoom: 1,
      offsetX: 0,
      offsetY: 0,
      regionX: 0,
      regionY: 0,
      regionWidth: dCrop.width,
      regionHeight: dCrop.height,
      regionKey: 0,
      defaultCropping: dCrop
    }
  },
  methods: {
    onResize(x, y, width, height) {
      this.regionX = x
      this.regionY = y
      this.regionWidth = width
      this.regionHeight = height
    },
    onDrag(x, y) {
      this.regionX = x
      this.regionY = y
    },
    resetView(){
      this.zoom = 1;
      this.offsetX = 0;
      this.offsetY = 0;
      this.regionX = this.defaultCropping.x;
      this.regionY = this.defaultCropping.y;
      this.regionWidth = this.defaultCropping.width;
      this.regionHeight = this.defaultCropping.height;
      this.regionKey++;
    },
    newImg(){
      this.resetView();

      this.axios.get(this.api).then((response) => {
        let imgURL = this.api;
        if(imgURL[imgURL.length-1] != "/"){ imgURL += "/" }
        imgURL += response.data;
        this.imgUrl = imgURL;

        this.axios.get(this.imgUrl, { responseType: 'arraybuffer' }).then((resp) => {
          let imgDat = Buffer.from(resp.data, 'binary').toString('base64')
          this.imgData = "data:image/png;base64,"+imgDat;
          this.instructionStep = 2;
        })

      })
    },
    changedAPI(){
      this.instructionStep = 1;
    },
    nextBtnClicked(){
      this.instructionStep++;
    },
    croppingIsDefault(cropping){
      if(
        cropping.x == this.defaultCropping.x
        && cropping.y == this.defaultCropping.y
        && cropping.width == this.defaultCropping.width
        && cropping.height == this.defaultCropping.height
      ){
        return true
      }
      return false
    },
    cropIsValid(cropping){
      if(cropping.x < 0
        || cropping.y < 0
        || (cropping.x+cropping.width) > this.$refs.imgRef.width
        || (cropping.y+cropping.height) > this.$refs.imgRef.height
        || this.croppingIsDefault(cropping)
      ){
        return false
      }

      return true
    },
    sendSelection(){
      let postData = {
        x: Math.round( this.regionX*(1/this.zoom) - this.offsetX*(this.zoom) ),
        y: Math.round( this.regionY*(1/this.zoom) - this.offsetY*(this.zoom) ),
        width: Math.round( this.regionWidth*(1/this.zoom) ),
        height: Math.round( this.regionHeight*(1/this.zoom) )
      }

      if(this.imgUrl == "" || !this.cropIsValid(postData)){
        return;
      }
      this.axios({
        method: 'post',
        url: this.imgUrl,
        data: postData
      }).then((response) => {
        if(response.status == 200){
          this.newImg();
        }
      });

    },
  }
};
</script>

<style scoped lang="scss">
.regionBox{
  border: 2px solid blue;
  background-color: rgba(100,100,255,0.15)
}
.cropperContainer {
  position: relative;
  .imageview{
    width: 600px;
    height: 600px;
    max-width: 95vw;
    border: 2px solid rgb(120,120,120);
    overflow: hidden;
    position: relative;
    display: flex;
    align-items: flex-start;
    justify-content: flex-start;
    .imgContainer {
      position: relative;
      overflow: hidden;
      width: 100%;
      height: 100%;
      box-sizing: border-box;
      display: flex;
      align-items: flex-start;
      justify-content: flex-start;
      img {
        transform-origin: 0% 0%;
        user-select: none;
      }
    }
    .scaleControls {
      position: absolute;
      top: 0;
      right: 0;
      display: flex;
      align-items: center;
      flex-direction: column;
      .moveContainer{
        margin-top: 3px;
        display: flex;
        flex-direction: row;
        .moveContainerColumn {
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          button{
            height: 1.5rem;
            width: 1.5rem;
            font-size: 1em;
          }
        }
      }
      .scaleControlsRow {
        display: flex;
      }
      input {
        width: 2em;
        height: 1rem;
        margin: 0 2px 0 2px;
        text-align: center;
      }
      button {
        border-radius: 25%;
        border: 1px solid gray;
        height: 1.4rem;
        width: 1.4rem;
        padding: 0;
        text-align: center;
        font-weight: bold;
        font-family: 'Courier New', monospace;
        cursor: pointer;
      }
    }
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
    display: flex;
    flex-direction: row;
    justify-content: center;
    > h4{
      margin: 0;
      color: rgb(60,60,65);
    }
  }

  .cpBtnLeft, .cpBtnRight {
    min-width: 8em;
  }
  .cpBtnLeft{
    display: flex;
    justify-content: flex-start;
  }
  .cpBtnRight{
    display: flex;
    justify-content: flex-end;
  }
}

.nextBtn{
  margin-left: 0.75em;
}

</style>
