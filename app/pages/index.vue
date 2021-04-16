<template>
  <div>
    <div id="cameraArea">
      <img v-if="code.length" src="" alt="result" class="resultImg" />
    </div>
    <p v-if="code.length" class="getMessage">取得できました</p>
    <p class="resultCode">{{ code }}</p>
    <button @click="startScan">Scan</button>
    <button aria-label="close" @click.prevent.stop="stopScan">
      Stop
    </button>
  </div>
</template>

<script >
import Vue from "vue";


export default Vue.extend({
  name: "BarcodeScanner",
  data() {
    return {
      Quagga: null,
      code: ""
    };
  },
  methods: {
    startScan() {
      this.code = "";
      this.initQuagga();
    },
    stopScan() {
      this.Quagga.offProcessed(this.onProcessed)
      this.Quagga.offDetected(this.onDetected)
      this.Quagga.stop();
    },
    initQuagga() {
      this.Quagga = require("quagga");
      this.Quagga.onProcessed(this.onProcessed);
      this.Quagga.onDetected(this.onDetected);

      // 設定
      const config = {
        inputStream: {
          name: "Live",
          type: "LiveStream",
          target: document.querySelector("#cameraArea"),
          constraints: { facingMode: "environment" }
        },
        numOfWorkers: navigator.hardwareConcurrency || 4,
        decoder: { readers: ["ean_reader", "ean_8_reader"] }
      };
      this.Quagga.init(config, this.onInit);
    },
    onInit(err) {
      if (err) {
        console.log(err);
        return;
      }
      console.info("Initialization finished. Ready to start");
      this.Quagga.start();
    },
    onDetected(success) {
      this.code = success.codeResult.code;
      // 取得時の画像を表示
      const $resultImg = document.querySelector(".resultImg");
      $resultImg.setAttribute("src", this.Quagga.canvas.dom.image.toDataURL());
      this.Quagga.stop();
    },
    onProcessed(result) {
      const drawingCtx = this.Quagga.canvas.ctx.overlay;
      const drawingCanvas = this.Quagga.canvas.dom.overlay;

      if (result) {
        // 検出中の緑の線の枠
        if (result.boxes) {
          drawingCtx.clearRect(0, 0, drawingCanvas.width, drawingCanvas.height);
          const hasNotRead = (box) => box !== result.box;
          result.boxes.filter(hasNotRead).forEach((box) => {
            this.Quagga.ImageDebug.drawPath(box, { x: 0, y: 1 }, drawingCtx, {
              color: "green",
              lineWidth: 2
            });
          });
        }

        // 検出に成功した瞬間の青い線の枠
        if (result.box) {
          this.Quagga.ImageDebug.drawPath(
            result.box,
            { x: 0, y: 1 },
            drawingCtx,
            {
              color: "blue",
              lineWidth: 2
            }
          );
        }

        // 検出に成功した瞬間の水平の赤い線
        if (result.codeResult && result.codeResult.code) {
          this.Quagga.ImageDebug.drawPath(
            result.line,
            { x: "x", y: "y" },
            drawingCtx,
            {
              color: "red",
              lineWidth: 3
            }
          );
        }
      }
    }
  }
});
</script>

<style>
#cameraArea {
  overflow: hidden;
  width: 320px;
  height: 240px;
  margin: auto;
  position: relative;
  display: flex;
  align-items: center;
}
#cameraArea video,
#cameraArea canvas {
  width: 320px;
  height: 240px;
}
button {
  width: 100px;
  height: 40px;
  background-color: #fff;
  border: 1px solid #333;
  margin-top: 30px;
}
.resultImg {
  width: 100%;
}
.resultCode {
  font-size: 24px;
  font-weight: bold;
  text-align: center;
}
.getMessage {
  color: red;
}
</style>