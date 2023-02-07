<template>
  <div id="canvasContainer">
    <canvas id="canvas"></canvas>
  </div>
</template>

<script>
import { fabric } from "fabric";
import FontFaceObserver from "fontfaceobserver";
import { copyImageToClipboard } from "copy-image-clipboard";
import { exportFile, Notify } from "quasar";

let fabricCanvas = null;

export default {
  name: "FabricCanvas",
  data() {
    return {
      background: null,
      exportBlob: null,
    };
  },
  methods: {
    init() {
      let that = this;

      let bodyWidth = this.$q.screen.width;
      let bodyHeight = this.$q.screen.height;

      fabricCanvas = new fabric.Canvas("canvas", {
        width: bodyWidth * 0.8,
        height: bodyHeight,
        backgroundColor: "rgba(0,0,0,0)",
      });

      this.$bus.on("canvas-refresh", (texts, image, advanced) => {
        that.generate(texts, advanced);
        that.setBackground(image);
      });

      this.$bus.on("canvas-export", (method) => {
        let backgroundImage = fabricCanvas["backgroundImage"];
        let bgWidth = backgroundImage["width"];
        let bgHeight = backgroundImage["height"];
        let scaleX = backgroundImage["scaleX"];
        let scaleY = backgroundImage["scaleY"];
        let left = backgroundImage["left"];
        let top = backgroundImage["top"];

        let finalBgWidth = bgWidth * scaleX;
        let finalBgHeight = bgHeight * scaleY;
        let finalLeft = left - finalBgWidth / 2;
        let finalTop = top - finalBgHeight / 2;

        let format = method;

        if (method === "clipboard") format = "jpeg";

        let dataURL = fabricCanvas.toDataURL({
          width: finalBgWidth,
          height: finalBgHeight,
          left: finalLeft,
          top: finalTop,
          format: format,
        });

        if (method === "countdown") {
          format = "png";
          dataURL = fabricCanvas.getObjects()[0].toDataURL({
            width: finalBgWidth,
            height: finalBgHeight,
            left: finalLeft,
            top: finalTop,
            format: format,
          });
        }

        if (method === "jpeg" || method === "png" || method === "countdown") {
          let dataUrlToBlob = require("dataurl-to-blob");
          let dataBlob = dataUrlToBlob(dataURL);

          const status = exportFile("countdown-image." + format, dataBlob);

          if (status)
            Notify.create({
              message: "已成功下载文件。",
              color: "positive",
              position: "top-left",
            });
          else
            Notify.create({
              message: "下载文件失败。",
              color: "negative",
              position: "top-left",
            });
        }

        if (method === "clipboard") {
          copyImageToClipboard(dataURL)
            .then(() => {
              Notify.create({
                message: "已复制到剪贴板。",
                color: "positive",
                position: "top-left",
              });
            })
            .catch(() => {
              Notify.create({
                message: "无法复制到剪贴板。",
                color: "negative",
                position: "top-left",
              });
            });
        }
      });
    },

    setBackground(uri) {
      let that = this;
      let domWidth = this.$q.screen.width * 0.8;
      let domHeight = this.$q.screen.height;

      fabric.Image.fromURL(uri, function (oImg) {
        let width = oImg.width;
        let height = oImg.height;

        if (height > width) oImg.scale(domHeight / height);
        else oImg.scale(domWidth / width);

        let center = fabricCanvas.getCenter();
        fabricCanvas.setBackgroundImage(
          oImg,
          fabricCanvas.renderAll.bind(fabricCanvas),
          {
            top: center.top,
            left: center.left,
            originX: "center",
            originY: "center",
          }
        );

        fabricCanvas.renderAll();
      });
    },
    generate(texts, advanced) {
      let that = this;

      let shadow = "rgba(0, 0, 0, 0.5) 0px 0px 15px";
      let fontCHNName = "字魂59号-创粗黑";
      let fontENGName = "Alte DIN 1451 Mittelschrift gepraegt Regular";

      let textColor = advanced["textColor"];
      let hintColor = advanced["hintColor"];

      let mainSpacing = advanced["mainSpacing"];
      let subSpacing = advanced["subSpacing"];

      let fontCHN = new FontFaceObserver(fontCHNName);
      let fontENG = new FontFaceObserver(fontENGName);

      Promise.all([fontCHN.load(), fontENG.load()]).then(function () {
        //左边长方形
        let rect = new fabric.Rect({
          top: 43,
          left: 0,
          width: 4,
          height: 68,
          fill: hintColor,
        });

        let rectRight = rect["left"] + rect["width"];

        //还剩
        let textStart = new fabric.Text(texts["start"].toString(), {
          top: 34,
          left: rectRight + 4,
          fontFamily: fontCHNName,
          fill: textColor,
          fontSize: 36,
          shadow: shadow,
          charSpacing: mainSpacing,
          lineHeight: 0.85,
        });

        let textStartRight = textStart["left"] + textStart["width"];

        //3
        let textCountdown = new fabric.Text(texts["countdown"].toString(), {
          top: -4,
          left: textStartRight + 4,
          fontFamily: fontENGName,
          fill: hintColor,
          fontSize: 84,
          shadow: shadow,
          charSpacing: 0,
          lineHeight: 0.85,
        });

        let textCountdownRight = textCountdown["left"] + textCountdown["width"];

        //秒
        let textEnd = new fabric.Text(texts["end"].toString(), {
          top: 34,
          left: textCountdownRight,
          fontFamily: fontCHNName,
          fill: textColor,
          fontSize: 36,
          shadow: shadow,
          charSpacing: mainSpacing,
          lineHeight: 0.85,
        });
        let textEndRight = textEnd["left"] + textEnd["width"];

        //SUN SUPERNOVA IN 3 SECONDS
        let textSub = new fabric.Text(texts["sub"].toString().toUpperCase(), {
          top: 76,
          left: rectRight + 6,
          fontFamily: fontENGName,
          fill: textColor,
          fontSize: 18,
          shadow: shadow,
          charSpacing: subSpacing,
          lineHeight: 0.85,
        });

        //距太阳超新星爆发
        let textTop = new fabric.Text(texts["top"].toString(), {
          top: 0,
          left: textStartRight + 2,
          fontFamily: fontCHNName,
          fill: textColor,
          fontSize: 36,
          shadow: shadow,
          textAlign: "right",
          originX: "right",
          charSpacing: mainSpacing,
          lineHeight: 0.85,
        });

        let fullWidth = textTop["width"] + textEndRight;

        let groupMain = new fabric.Group(
          [rect, textTop, textStart, textCountdown, textEnd, textSub],
          {
            top: 100,
            left: 100,
          }
        );

        groupMain.setControlVisible("ml", false);
        groupMain.setControlVisible("mb", false);
        groupMain.setControlVisible("mr", false);
        groupMain.setControlVisible("mt", false);
        groupMain.setControlVisible("mtr", false);

        fabricCanvas.add(groupMain);

        that.updateGroupProps();
      });
    },
    updateGroupProps() {
      if (fabricCanvas.getObjects().length <= 1) return;

      let group = fabricCanvas.getObjects()[0];
      let top = group["top"];
      let left = group["left"];
      let zoomX = group["zoomX"];
      let zoomY = group["zoomY"];
      let scaleX = group["scaleX"];
      let scaleY = group["scaleY"];

      fabricCanvas.remove(fabricCanvas.getObjects()[0]);

      fabricCanvas.getObjects()[0].set({
        top: top,
        left: left,
        zoomX: zoomX,
        zoomY: zoomY,
        scaleX: scaleX,
        scaleY: scaleY,
      });
      fabricCanvas.getObjects()[0].setCoords();
    },
  },
  mounted() {
    this.init();

    window.onresize = () => {
      // this.$q.screen 刷新不及时
      // this.$q.screen refreshes with delay
      fabricCanvas.setWidth(document.body.clientWidth * 0.8);
      fabricCanvas.setHeight(document.body.clientHeight);
    };
  },
};
</script>

<style scoped>
#canvasContainer {
  background-image: url(data:image/gif;base64,R0lGODlhCgAKAIAAAOLi4v///yH5BAAHAP8ALAAAAAAKAAoAAAIRhB2ZhxoM3GMSykqd1VltzxQAOw==);
  display: inline-block;
}

/* #canvasContainer {
  width: 100%;
  height: 100%;
}

#canvas {
  width: 100%;
  height: 100%;
} */
</style>
