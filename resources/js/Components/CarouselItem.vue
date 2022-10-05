<template>
  <div class="static">
    <canvas height="600" width="400" ref="carouselItem"></canvas>
    <div class="absolute bottom-0 left-0" v-if="props.hasUploaded"><img style="width: 50px; height: 50px"
                                       src="https://www.citypng.com/public/uploads/preview/-31622653611o9op4nhtyt.png">
    </div>
  </div>

</template>

<script setup>
import {onMounted, ref, watch,} from 'vue'

const carouselItem = ref()
const props = defineProps(['imageCanvas', 'hasUploaded']);


onMounted(() => {
  const context = carouselItem.value.getContext('2d');
  context.drawImage(props.imageCanvas, 0, 0);
})
</script>