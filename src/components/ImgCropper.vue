<template>
  <div class="image-cropper" ref="imageCropperRef" @mouseover="handleMouseOver" @mouseout="handleMouseOut">

      <div class="img-cropper__canvas">
        <img :src="cropperModel.url" alt="" />
      </div>
      <div class="img-cropper__cut" :style="{
        'width': cutModel.width + 'px',
        'height': cutModel.height + 'px',
        'top': cutModel.y + 'px',
        'left': cutModel.x + 'px'
      }">
      <ul class="img-cropper__cut__handle">
        <li class="img-cropper__cut__left_top square_blue"></li>
        <li class="img-cropper__cut__top square_blue"></li>
        <li class="img-cropper__cut__right_top square_blue"></li>
        <li class="img-cropper__cut__left square_blue"></li>
        <li class="img-cropper__cut__right square_blue"></li>
        <li class="img-cropper__cut__left_bottom square_blue"></li>
        <li class="img-cropper__cut__bottom"></li>
        <li class="img-cropper__cut__right_bottom square_blue"></li>

        <li class="img-cropper__cut__line_top"></li>
        <li class="img-cropper__cut__line_right"></li>
        <li class="img-cropper__cut__line_bottom"></li>
        <li class="img-cropper__cut__line_left"></li>
      </ul>

      </div>
      <div class="img-cropper__drop"></div>
      <div class="img-cropper__modal"></div>
 
  
  </div>

</template>
<script setup lang="ts">
import { ref,reactive,watch } from 'vue'

type ImgType = String | Blob | null | File

const imageCropperRef = ref(null)

const props = defineProps({
  img: {
    type: [String,Blob,null,File],
    default: ''
  },
  canvasHeight: {
    type: [Number],
    default:0
  },
  canvasWidth: {
    type: [Number],
    default:0
  }
})

const cropperModel = reactive({
  img: '',
  url:'',
  loading: false,
  scale: 1,
  rotate: 0,
  orientation:1

})

const cutModel = reactive({
  width:0,
  height:0,
  x:0,
  y:0
})




const handleImgChange = () => {

  
  if (!props.img) {
    handleClearCropper()
    return

  }

  console.log('img changed',props.img)
  cropperModel.loading = true
  cropperModel.scale = 1
  cropperModel.rotate = 0

  const img = new Image()

  img.onload = async () => {
    let width = img.width
    let height = img.height
    // const imageData = await getImageData(img)
    // cropperModel.orientation = imageData.orientation 
    const maxWidth = imageCropperRef.value.offsetWidth
    console.log(maxWidth,'maxWidth')
    if(width > maxWidth) {
      const rate = height /width 
      width = maxWidth 
      height = rate * width
    }

    // cropperModel.img = img 
    // cropperModel.imgHeight = height
    // cropperModel.imgWidth = width
    drawImageToCanvas(img, width, height)

  }



  img.onerror = (error) => {
    console.log('onerror',error)
    cropperModel.loading = false
  }



  if(props.img instanceof File ) {

    if(!props.img.type.indexOf('image') === 0) { 
      console.log('not image')
      return
    }
    const reader = new FileReader()
    reader.onload = () => {

      // img.src = reader.result
      const arrayBuffer = reader.result
      const blob = new Blob([arrayBuffer],{type: props.img?.type || 'image/png'})
      const URL = window.URL || window.webkitURL
      const src = URL.createObjectURL(blob)
      img.src = src


    }
    reader.onerror = (error) => {
      console.error('FileReader',error)
    }
    // // 转换为base64
    // // render.readAsDataURL(props.img)

    // 转换为arrayBuffer
    reader.readAsArrayBuffer(props.img)
  }else {
    img.src = props.img
  }



}
watch(() => props.img, handleImgChange)

const drawImageToCanvas = (img:Image,width:number,height:number) => {
  console.log('drawImageToCanvas',img)
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  ctx.save() 

  canvas.width = width 
  canvas.height = height

  ctx.drawImage(img, 0, 0, width, height)
  ctx.restore()

  canvas.toBlob(blob => {
    const url = URL.createObjectURL(blob)
    cropperModel.url = url

  })

}


const handleClearCropper = () => {
  cropperModel.img = ''
}




const handleMouseOver = () => {
  console.log('mouseover')
}

const handleMouseOut = () => {
  console.log('mouseout')
}



const getImageData = (img: ImgType): Promise => new Promise(async (resolve, reject) => {
  console.log(img,'img')

  const obj = {}

  const DataURLReg = /^(data:)/
  const BlobURLReg = /^(blob:)/
  const NetworkURLReg = /^(https?|ftp|file):\/\/|^\//


  if (img.src) {
    try {
      switch (img.src) {
        case DataURLReg.test(img.src):
          obj.arrayBuffer = await base64ToArrayBuffer(img.src)
          break
        case BlobURLReg.test(img.src):
          obj.arrayBuffer = await blobToArrayBuffer(img.src)
          break
        case NetworkURLReg.test(img.src):
          obj.arrayBuffer = await networkImgToArrayBuffer(img.src)
          break
        default:
          break
      }
      obj.orientation = await getOrientation(obj.arrayBuffer)
    } catch (error) {
      reject(error)
    }

  }
})

/**
 * @description: 将base64转换为ArrayBuffer
 * @param {*} base64 
 * @param {*} reject 
 * @return {*} Promise<Uint8Array>
 */
const base64ToArrayBuffer = (base64: base64): Promise<Uint8Array> => new Promise((resolve, reject) => {
  const binary = atob(base64)
  const array = new Uint8Array(binary.length)
  for (let i = 0; i < binary.length; i++) {
    array[i] = binary.charCodeAt(i)
  }
  resolve(array)
})

/**
 * @description: 将网络图片转换为ArrayBuffer
 * @param {*} url 
 * @param {*} reject
 * @return {*}
 */
const networkImgToArrayBuffer = (url: string): Promise<Uint8Array> => new Promise((resolve, reject) => {
  const http = new XMLHttpRequest()
  http.open('GET', url, true)
  http.send()
  http.onload = () => {
    const arrayBuffer = http.response
    resolve(arrayBuffer)
  }
})

/**
 * @description: 将blob转换为ArrayBuffer
 * @param {*} blob
 * @param {*} reject
 * @return {*}
 */
const blobToArrayBuffer = (blob: Blob): Promise<Uint8Array> => new Promise((resolve, reject) => {
  const reader = new FileReader()
  reader.readAsArrayBuffer(blob)
  reader.onload = () => {
    const arrayBuffer = reader.result
    resolve(arrayBuffer)
  }
})

/**
 * @description: 获取图片方向
 * @param {*} arrayBuffer
 * @param {*} reject
 * @return {*}
 */
const getOrientation = (arrayBuffer: Uint8Array): Promise<number> => new Promise((resolve, reject) => {

  
})






</script>

<style scoped lang="scss">
$crop_blue: #16B5FF;
.image-cropper {
  width: 80%;
  .img-cropper__canvas {
    overflow: hidden;
  }
  .img-cropper__cut {
    position: relative;
    outline: 1px solid $crop_blue;
    outline-color: $crop_blue;
    >li {
      list-style: none;
    }
    .img-cropper__cut__handle {
     
    }
    .square_blue {
      width: 8px;
      height: 8px;
      color: $crop_blue;
    }
  }
}
</style>
