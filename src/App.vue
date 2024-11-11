

<template>
  <div class="app-page">
    <!--  -->
    <ImgCropper :img="file"/>

    <input type="file" @change="handleUpdateFIle">
  </div>

</template>
<script setup lang="ts">
import { ref } from 'vue'
import ImgCropper from './components/ImgCropper.vue'

const file = ref(null)

const handleUpdateFIle = (e: file) => {
  file.value = e.target.files[0]
  console.log(e,'file')
}

</script>
<style scoped>

</style>
