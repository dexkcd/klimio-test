<template>
  <Carousel ref="myCarousel" :settings="settings" :breakpoints="breakpoints">
    <Slide v-for="(imageCanvas,index) in pictureList" :key="index" ref="pictureListItemsRef">
      <div class="carousel__item">
        {{ index }}
        <CarouselItem :imageCanvas="imageCanvas" />
      </div>
    </Slide>

    <template #addons>
      <Navigation/>
    </template>
  </Carousel>
</template>
<style>
.carousel__item {
  width: 100%;

}
</style>
<script setup>
import {onMounted, reactive, ref} from 'vue'
import {Carousel, Navigation, Slide} from 'vue3-carousel'

import 'vue3-carousel/dist/carousel.css'
import CarouselItem from "@/Components/CarouselItem.vue";

const props = defineProps(['pictureList']);
const pictureListItemsRef = ref([])

const myCarousel = ref();

// carousel settings
const settings = reactive({
  itemsToShow: 1,
  snapAlign: 'center'
});

// breakpoints are mobile first
// any settings not specified will fallback to the carousel settings
const breakpoints = reactive({
  // 700px and up
  700: {
    itemsToShow: 3.5,
    snapAlign: 'center',
  },
  // 1024 and up
  1024: {
    itemsToShow: 5,
    snapAlign: 'start',
  },
});

onMounted(()=>{
})
</script>