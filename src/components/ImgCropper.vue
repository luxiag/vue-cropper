<template>
  <div class="image-cropper" ref="imageCropperRef">

    <div class="img-cropper__canvas">
      <img :src="cropperModel.url" alt="" />
    </div>
    <div class="img-cropper__cut"       :style="{
      'width': cutModel.width + 'px',
      'height': cutModel.height + 'px',
      'transform':`translate(${cutModel.x}px, ${cutModel.y}px)`

    }" 

    > 
      <div class="img-cropper__cut__move"      @mousedown="handleCutModelMoveDownEvent" 
      @touchstart="handleCutModelMoveDownEvent"></div>
      <ul class="img-cropper__cut__handle">
        <li class="img-cropper__cut__left_top square_blue"  @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.left_top)"></li>
        <li class="img-cropper__cut__top square_blue" @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.top)"></li>
        <li class="img-cropper__cut__right_top square_blue"  @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.right_top)"></li>
        <li class="img-cropper__cut__left square_blue"  @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.left)"></li>
        <li class="img-cropper__cut__right square_blue"  @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.right)"></li>
        <li class="img-cropper__cut__left_bottom square_blue"  @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.left_bottom)"></li>
        <li class="img-cropper__cut__bottom square_blue"  @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.bottom)"></li>
        <li class="img-cropper__cut__right_bottom square_blue"  @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.right_bottom)"></li>

        <li class="img-cropper__cut__line_top" @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.top)"></li>
        <li class="img-cropper__cut__line_right" @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.right)"></li>
        <li class="img-cropper__cut__line_bottom" @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.bottom)"></li>
        <li class="img-cropper__cut__line_left" @mousedown="$e =>handleCutModelSizeEvent($e,MoveTypeEnum.left)"></li>
      </ul>

    </div>
    <div class="img-cropper__drop"></div>
    <div class="img-cropper__modal"></div>


  </div>

</template>
<script setup lang="ts">
import { ref, reactive, watch, onMounted, nextTick } from 'vue'

type ImgType = String | Blob | null | File

const imageCropperRef = ref<HTMLElement>()

const props = defineProps({
  img: {
    type: [String, Blob, null, File],
    default: ''
  },
  canvasHeight: {
    type: [Number],
    default: 0
  },
  canvasWidth: {
    type: [Number],
    default: 0
  }
})

const cropperModel = reactive({
  img: '',
  url: '',
  loading: false,
  scale: 1,
  rotate: 0,
  orientation: 1

})


// 剪切框

enum MoveTypeEnum {
  left_top = 'left_top',
  top = 'top',
  right_top = 'right_top',
  left = 'left',
  right = 'right',
  left_bottom = 'left_bottom',
  bottom = 'bottom',
  right_bottom = 'right_bottom'
}

const cutModel = reactive({
  maxWidth: 0,
  maxHeight: 0,
  minWidth:10,
  minHeight:10,
  width: 0,
  height: 0,

  x: 0,
  y: 0,
  // start move point
  startMoveX:0,
  startMoveY:0,
  beforeMoveX:0,
  beforeMoveY:0,
  beforeChangeWidth:0,
  beforeChangeHeight:0,
  moveType: MoveTypeEnum.left_top
})




type MouseEventType = MouseEvent | TouchEvent

const handleCutModelMoveDownEvent = (e: MouseEventType) => {
  e.preventDefault()
  // 双指触控
  if('touches' in e&&e.touches.length>1){ 
    return
  }


  window.addEventListener('mousemove', moveCutModelEvent)
  window.addEventListener('mouseup',leaveCutModelEvent)
  window.addEventListener('touchmove', moveCutModelEvent)
  window.addEventListener('touchend', leaveCutModelEvent)

  cutModel.startMoveX = 'clientX' in e ? e.clientX : 'touches' in e ? e.touches[0].clientX : 0
  cutModel.startMoveY = 'clientY' in e ? e.clientY : 'touches' in e ? e.touches[0].clientY : 0

  cutModel.beforeMoveX = cutModel.x
  cutModel.beforeMoveY = cutModel.y
  
}

const moveCutModelEvent = (e: MouseEventType) => {
 e.preventDefault()

  const movingX = 'clientX' in e ? e.clientX : 'touches' in e ? e.touches[0].clientX : 0
  const movingY = 'clientY' in e ? e.clientY : 'touches' in e ? e.touches[0].clientY : 0

  const lengthX =movingX - cutModel.startMoveX + cutModel.beforeMoveX
  const lengthY =movingY - cutModel.startMoveY + cutModel.beforeMoveY
  nextTick(()=>{
    cutModel.x = lengthX < 0 ? 0 : lengthX > cutModel.maxWidth - cutModel.width ? cutModel.maxWidth - cutModel.width : lengthX;
    cutModel.y = lengthY < 0 ? 0 : lengthY > cutModel.maxHeight - cutModel.height ? cutModel.maxHeight - cutModel.height : lengthY;
  })

  
}

const leaveCutModelEvent = (e:MouseEventType) => {
  window.removeEventListener('mousemove', moveCutModelEvent)
  window.removeEventListener('mouseup', leaveCutModelEvent)
  window.removeEventListener('touchmove', moveCutModelEvent)
  window.removeEventListener('touchend', leaveCutModelEvent)
}


const handleCutModelSizeEvent = (e:MouseEventType,type:MoveTypeEnum ) => {
  e.preventDefault()
  if('touches' in e&&e.touches.length>1){ 
    return
  }
  cutModel.startMoveX = 'clientX' in e ? e.clientX : 'touches' in e ? e.touches[0].clientX : 0
  cutModel.startMoveY = 'clientY' in e ? e.clientY : 'touches' in e ? e.touches[0].clientY : 0
  cutModel.beforeMoveX = cutModel.x
  cutModel.beforeMoveY = cutModel.y
  cutModel.beforeChangeHeight = cutModel.height
  cutModel.beforeChangeWidth = cutModel.width
  window.addEventListener('mousemove',changeCutModelSize)
  window.addEventListener('mouseup',leaveCutModelSize)
  window.addEventListener('touchmove',changeCutModelSize)
  window.addEventListener('touchend',leaveCutModelSize)
  cutModel.moveType = type
}

const changeCutModelSize = (e: MouseEventType) => {
  e.preventDefault()

  const movingX = 'clientX' in e ? e.clientX : 'touches' in e ? e.touches[0].clientX : 0
  const movingY = 'clientY' in e ? e.clientY : 'touches' in e ? e.touches[0].clientY : 0
  const height = cutModel.startMoveY - movingY 
  const width = cutModel.startMoveX - movingX
  switch(cutModel.moveType) { 
    case MoveTypeEnum.top:
      
      cutModel.height = 
        cutModel.beforeChangeHeight + height > cutModel.maxHeight - cutModel.beforeMoveY  ? cutModel.maxHeight - cutModel.beforeMoveY:
        cutModel.beforeChangeHeight + height < cutModel.minHeight ? cutModel.minHeight : 
        cutModel.beforeChangeHeight + height

      cutModel.y = cutModel.beforeMoveY - height < 0 ? 0 : 
        cutModel.beforeMoveY - height > cutModel.maxHeight ? cutModel.maxHeight: 
        cutModel.beforeMoveY - height    
    
      break;
    case MoveTypeEnum.bottom:
      cutModel.height = cutModel.y + cutModel.height >= cutModel.maxHeight ? cutModel.height: 
        cutModel.beforeChangeHeight - height > cutModel.maxHeight ? cutModel.maxHeight :
        cutModel.beforeChangeHeight - height < cutModel.minHeight ? cutModel.minHeight : 
        cutModel.beforeChangeHeight - height
      break;
    case MoveTypeEnum.left:

      cutModel.width =cutModel.beforeMoveX - width <= 0 ? cutModel.width:  
      cutModel.beforeChangeWidth + width > cutModel.maxWidth ? cutModel.maxWidth : 
      cutModel.beforeChangeWidth + width < cutModel.minWidth ? cutModel.minWidth : 
      cutModel.beforeChangeWidth + width

      cutModel.x = cutModel.beforeMoveX - width <= 0 ? 0 : 
      cutModel.beforeMoveX - width > cutModel.maxWidth ? cutModel.maxWidth: 
      cutModel.beforeMoveX - width
      break
    case MoveTypeEnum.right:

      cutModel.width = cutModel.width + cutModel.x >= cutModel.maxWidth ? cutModel.width : 
        cutModel.beforeChangeWidth - width > cutModel.maxWidth ? cutModel.maxWidth : 
        cutModel.beforeChangeWidth - width < cutModel.minWidth ? cutModel.minWidth : 
        cutModel.beforeChangeWidth - width

      break
    case MoveTypeEnum.left_top:
      cutModel.width = cutModel.beforeChangeWidth + width > cutModel.maxWidth ? cutModel.maxWidth : cutModel.beforeChangeWidth + width < cutModel.minWidth ? cutModel.minWidth : cutModel.beforeChangeWidth + width
      cutModel.x = cutModel.beforeMoveX - width < 0 ? 0 : cutModel.beforeMoveX - width > cutModel.maxWidth ? cutModel.maxWidth: cutModel.beforeMoveX - width
      cutModel.height = cutModel.beforeChangeHeight + height > cutModel.maxHeight ? cutModel.maxHeight : cutModel.beforeChangeHeight + height < cutModel.minHeight ? cutModel.minHeight : cutModel.beforeChangeHeight + height
      cutModel.y = cutModel.beforeMoveY - height < 0 ? 0 : cutModel.beforeMoveY - height > cutModel.maxHeight ? cutModel.maxHeight: cutModel.beforeMoveY - height
      break
    case MoveTypeEnum.right_top:
    case MoveTypeEnum.left_bottom:
    case MoveTypeEnum.right_bottom:

  }


}


const leaveCutModelSize = (e:MouseEventType) => {

  window.removeEventListener('mousemove', changeCutModelSize)
  window.removeEventListener('mouseup', leaveCutModelSize)
  window.removeEventListener('touchmove', changeCutModelSize)
  window.removeEventListener('touchend', leaveCutModelSize)
}



const initCutModel = () => {
  const width = imageCropperRef.value?.offsetWidth || 500
  cutModel.maxWidth = width
  cutModel.maxHeight = width
  cutModel.width = width -200
  cutModel.height = width -200
  cutModel.x = 0
  cutModel.y = 0
}

onMounted(()=>{
  initCutModel()
})

const handleImgChange = () => {


  if (!props.img) {
    handleClearCropper()
    return

  }

  console.log('img changed', props.img)
  cropperModel.loading = true
  cropperModel.scale = 1
  cropperModel.rotate = 0

  const img = new Image()

  img.onload = async () => {
    let width = img.width
    let height = img.height
    // const imageData = await getImageData(img)
    // cropperModel.orientation = imageData.orientation 
    const maxWidth = imageCropperRef.value?.offsetWidth || 0
    console.log(maxWidth, 'maxWidth')
    if (width > maxWidth && maxWidth !== undefined) {
      const rate = height / width
      width = maxWidth
      height = rate * width
    }

    // cropperModel.img = img 
    // cropperModel.imgHeight = height
    // cropperModel.imgWidth = width
    drawImageToCanvas(img, width, height)

  }



  img.onerror = (error) => {
    console.log('onerror', error)
    cropperModel.loading = false
  }



  if (props.img instanceof File) {

    if (props.img.type.indexOf('image') !== 0) {
      console.log('not image')
      return
    }
    const reader = new FileReader()
    reader.onload = () => {

      // img.src = reader.result
      const arrayBuffer = reader.result
      if (!arrayBuffer) return
      if (props.img instanceof File) {
        const blob = new Blob([arrayBuffer], { type: props.img.type || 'image/png' })
        const URL = window.URL || window.webkitURL
        const src = URL.createObjectURL(blob)
        img.src = src
      }else {
        console.error('props.img is not File')
      }



    }
    reader.onerror = (error) => {
      console.error('FileReader', error)
    }
    // // 转换为base64
    // // render.readAsDataURL(props.img)

    // 转换为arrayBuffer
    reader.readAsArrayBuffer(props.img)
  } else if (typeof props.img === 'string') {
    img.src = props.img
  }



}
watch(() => props.img, handleImgChange)

const drawImageToCanvas = (img: HTMLImageElement, width: number, height: number) => {
  console.log('drawImageToCanvas', img)
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  if (!ctx) return
  ctx.save()

  canvas.width = width
  canvas.height = height

  ctx.drawImage(img, 0, 0, width, height)
  ctx.restore()

  canvas.toBlob(blob => {
    if (!blob) {
      console.error('blob is null')
      return
    }
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



const getImageData = (img: ImgType) => new Promise(async (resolve, reject) => {
  console.log(img, 'img')

  const obj = {}

  const DataURLReg = /^(data:)/
  const BlobURLReg = /^(blob:)/
  const NetworkURLReg = /^(https?|ftp|file):\/\/|^\//


  // if (img.src) {
  //   try {
  //     switch (img.src) {
  //       case DataURLReg.test(img.src):
  //         obj.arrayBuffer = await base64ToArrayBuffer(img.src)
  //         break
  //       case BlobURLReg.test(img.src):
  //         obj.arrayBuffer = await blobToArrayBuffer(img.src)
  //         break
  //       case NetworkURLReg.test(img.src):
  //         obj.arrayBuffer = await networkImgToArrayBuffer(img.src)
  //         break
  //       default:
  //         break
  //     }
  //     obj.orientation = await getOrientation(obj.arrayBuffer)
  //   } catch (error) {
  //     reject(error)
  //   }

  // }
})

/**
 * @description: 将base64转换为ArrayBuffer
 * @param {*} base64 
 * @param {*} reject 
 * @return {*} Promise<Uint8Array>
 */
const base64ToArrayBuffer = (base64: string): Promise<Uint8Array> => new Promise((resolve, reject) => {
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
    // resolve(arrayBuffer)
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

$square_size: 8px;
$squire_line_size: 1px;

$drop_range: 10px;

.image-cropper {
  width: 500px;
  height: 500px;
  margin: auto;
  position: relative;

  .img-cropper__canvas {
    overflow: hidden;
    height: 100%;
    width: 100%;
  }

  .img-cropper__cut {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    cursor: move;
    outline: 1px solid $crop_blue;
    outline-color: $crop_blue;



    .img-cropper__cut__move {
      width: 100%;
      height: 100%;
      z-index: 1;
      margin: $drop_range/2;
      box-sizing: border-box;
    }
    .img-cropper__cut__handle {
      list-style-type: none;
      z-index: 11;
      padding: 0;
      margin: 0;
      >li {
        list-style-type: none;
      }

      .square_blue {
        width: $square_size;
        height: $square_size;
        background-color: $crop_blue;
      }
      .img-cropper__cut__left_top {
        position: absolute;
        left: -$square_size/2;
        top: -$square_size/2;
        cursor: nw-resize;
      }
      .img-cropper__cut__top {
        position: absolute;
        top: 0;
        left: 50%;
        margin-left: -$square_size/2;
        margin-top: -$square_size/2;
        cursor: n-resize;
      }
      .img-cropper__cut__right_top {
        position: absolute;
        right: -$square_size/2;
        top: -$square_size/2;
        cursor: ne-resize;
      }
      .img-cropper__cut__left {
        position: absolute;
        left: -$square_size/2;
        top: 50%;
        margin-top: -$square_size/2;
      }
      .img-cropper__cut__right {
        position: absolute;
        right: -$square_size/2;
        top: 50%;
        margin-top: -$square_size/2;
      }
      .img-cropper__cut__left_bottom {
        position: absolute;
        left: -$square_size/2;
        bottom: -$square_size/2;
        cursor: sw-resize;
      }
      .img-cropper__cut__bottom {
        position: absolute;
        left: 50%;
        margin-left: -$square_size/2;
        bottom: -$square_size/2;
        cursor: s-resize;
      }
      .img-cropper__cut__right_bottom {
        position: absolute;
        right: -$square_size/2;
        bottom: -$square_size/2;
        cursor: se-resize;
      }

      .img-cropper__cut__line_top {
        height: $drop_range;
        position: absolute;
        width: 100%;
        top: 0px;
        left: 0px;
        cursor: n-resize;
      }
      // line
      .img-cropper__cut__line_top {
        height: $drop_range;
        position: absolute;
        width: 100%;
        top: -$drop_range/2;
        left: 0px;
        cursor: n-resize;
      }
      .img-cropper__cut__line_bottom {
        height: $drop_range;
        position: absolute;
        width: 100%;
        bottom: 0px;
        left: 0px;
        cursor: s-resize;
      }
      .img-cropper__cut__line_left {
        height: 100%;
        position: absolute;
        width: $drop_range;
        top: 0px;
        left: 0px;
        cursor: w-resize;
      }
      .img-cropper__cut__line_right {
        height: 100%;
        position: absolute;
        width: $drop_range;
        top: 0px;
        right: 0px;
        cursor: e-resize;
      }


    }


  }
  .img-cropper__drop {
    height: 100%;
    width: 100%;
  }
}
</style>
