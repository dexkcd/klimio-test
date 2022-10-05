<template>
  <canvas ref="carouselItem"></canvas>
</template>

<script setup>
import {onMounted, onUpdated, reactive, ref, watch, watchEffect} from 'vue'
import {Carousel, Navigation, Slide} from 'vue3-carousel'

const carouselItem = ref()
const props = defineProps(['imageCanvas']);

onMounted(()=>{
    const context = carouselItem.value.getContext('2d');
    context.drawImage(props.imageCanvas, 0,0)
})
</script>