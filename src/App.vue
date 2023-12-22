<template>
  <div id="app">
    <div style="background: #fff">
      <vue-signature-pad id="signature" width="95%" height="300px" ref="signaturePad" :options="options" />
    </div>

    <div v-for="(item, index) in imgList" :key="index">
      <img :src="item.src" alt="" width="100" />
    </div>

    <div class="buttons">
      <button @click="save" class="btn">保存</button>
      <button @click="resume" class="btn">重置</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      options: {
        penColor: '#000',
        minWidth:'3',
        maxWidth:'3'
      },
      imgList: [],
      fileList: [],
    }
  },
  methods: {
    save () {
      const { data } = this.$refs.signaturePad.saveSignature()

      this.rotateBase64Img(data, 90, (res) => {
        console.log(res) // 旋转后的base64图片src
        this.fileList.push({
          file: this.dataURLtoFile(res, 'sign'),
          name: 'sign',
        })
        this.imgList.push({
          src: res,
        })
      })
    },

    // 清除重置
    resume () {
      this.$refs.signaturePad.clearSignature()
    },

    // 将base64转换为文件
    dataURLtoFile (dataurl, filename) {
      var arr = dataurl.split(','),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n)

      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }

      return new File([u8arr], filename, { type: mime })
    },

    // 通过canvas旋转图片
    rotateBase64Img (src, edg, callback) {
      var canvas = document.createElement('canvas')
      var ctx = canvas.getContext('2d')

      var imgW //图片宽度
      var imgH //图片高度
      var size //canvas初始大小

      if (edg % 90 != 0) {
        console.error('旋转角度必须是90的倍数!')
        throw '旋转角度必须是90的倍数!'
      }
      edg < 0 && (edg = (edg % 360) + 360)
      const quadrant = (edg / 90) % 4 //旋转象限
      const cutCoor = { sx: 0, sy: 0, ex: 0, ey: 0 } //裁剪坐标

      var image = new Image()

      image.crossOrigin = 'anonymous'
      image.src = src

      image.onload = function () {
        imgW = image.width
        imgH = image.height
        size = imgW > imgH ? imgW : imgH

        canvas.width = size * 2
        canvas.height = size * 2
        switch (quadrant) {
          case 0:
            cutCoor.sx = size
            cutCoor.sy = size
            cutCoor.ex = size + imgW
            cutCoor.ey = size + imgH
            break
          case 1:
            cutCoor.sx = size - imgH
            cutCoor.sy = size
            cutCoor.ex = size
            cutCoor.ey = size + imgW
            break
          case 2:
            cutCoor.sx = size - imgW
            cutCoor.sy = size - imgH
            cutCoor.ex = size
            cutCoor.ey = size
            break
          case 3:
            cutCoor.sx = size
            cutCoor.sy = size - imgW
            cutCoor.ex = size + imgH
            cutCoor.ey = size + imgW
            break
        }

        ctx.translate(size, size)
        ctx.rotate((edg * Math.PI) / 180)
        ctx.drawImage(image, 0, 0)

        var imgData = ctx.getImageData(
          cutCoor.sx,
          cutCoor.sy,
          cutCoor.ex,
          cutCoor.ey
        )

        if (quadrant % 2 == 0) {
          canvas.width = imgW
          canvas.height = imgH
        } else {
          canvas.width = imgH
          canvas.height = imgW
        }
        ctx.putImageData(imgData, 0, 0)
        callback(canvas.toDataURL())
      }
    },
  },
}
</script>

<style lang="scss">
html,
body {
  padding: 0;
  margin: 0;
}

#app {
  width: 100vw;
  height: 100vh;
  background: #ececec;
}

.btn {
  width: 35%;
  color: #fff;
  background: #5daaf3;
  border: none;
  height: 40px;
  border-radius: 20px;
  margin-top: 20px;
  margin-left: 40px;
}
</style>