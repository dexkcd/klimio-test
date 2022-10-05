<template>
  <canvas height="600" width="400" ref="carouselItem"></canvas>
</template>

<script setup>
import {onMounted, ref, } from 'vue'

const carouselItem = ref()
const props = defineProps(['imageCanvas']);

onMounted(()=>{
    const context = carouselItem.value.getContext('2d');
    context.drawImage(props.imageCanvas, 0,0)
})
</script>