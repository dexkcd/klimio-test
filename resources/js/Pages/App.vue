<script setup>
import {ref, onMounted, reactive} from 'vue';
import ImageCarousel from "@/Components/ImageCarousel.vue";
import PrimaryButton from "@/Components/PrimaryButton.vue";

const state = reactive({
  longitude: null,
  latitude: null,
  devices: null,
  activeDevice: null,
  isCameraOpen: false,
  isPhotoTaken: false,
  isShotPhoto: false,
  isLoading: false,
  link: '#',
  pictureList: []
})

const camera = ref()
const canvas = ref()


function toggleCamera() {
  if (state.isCameraOpen) {
    state.isCameraOpen = false;
    state.isPhotoTaken = false;
    state.isShotPhoto = false;
    stopCameraStream();
  } else {
    state.isCameraOpen = true;
    createCameraElement();
  }
}

function createCameraElement() {
  state.isLoading = true;

  const constraints = (window.constraints = {
    audio: false,
    video: {
      width: { ideal: 400 },
      height: { ideal: 600 }
    }
  });

  navigator.mediaDevices
      .getUserMedia(constraints)
      .then(stream => {
        state.isLoading = false;
        camera.value.srcObject = stream;
      })
      .catch(error => {
        console.log(error)
        state.isLoading = false;
        alert("Maybe the browser didn't support or there is some errors.");
      });
}

function stopCameraStream() {
  let tracks = camera.value.srcObject.getTracks();

  tracks.forEach(track => {
    track.stop();
  });
}

function takePhoto() {
  if (!state.isPhotoTaken) {
    state.isShotPhoto = true;
    const FLASH_TIMEOUT = 50;

    setTimeout(() => {
      state.isShotPhoto = false;
    }, FLASH_TIMEOUT);
  }

  state.isPhotoTaken = !state.isPhotoTaken;

  const context = canvas.value.getContext('2d');
  context.drawImage(camera.value, 0, 0, 400, 600);
  state.pictureList.push(canvas.value);
  uploadPicture()
}

async function uploadPicture() {

  canvas.value.toBlob(async (blob) => {
    let formData = new FormData();
    formData.append("file", blob);
    formData.append("filename", Date.now() + (Math.random() * 1000));
    axios.post('/api/picture', formData, {
      headers: {
        "Content-Type": "multipart/form-data",
      }

    }).then((res) => {
    }).catch((error) => {
      console.log(error)
    });


  }, "image/jpeg", 1)
}


onMounted(() => {
  window.navigator.geolocation
      .getCurrentPosition((geolocation) => {
        state.longitude = geolocation.coords.longitude
        state.latitude = geolocation.coords.latitude
      }, console.log);

})

</script>

<template>
  <div class="container mx-auto">

    <div class="grid grid-flow-row p-8">
      <div class="columns-2">
        <div class="">
          <PrimaryButton class="ml-4"
                  :class="{ 'is-primary' : !state.isCameraOpen, 'is-danger' : state.isCameraOpen}" @click="toggleCamera">
            <span v-if="!state.isCameraOpen">Open Camera</span>
            <span v-else>Close Camera</span>
          </PrimaryButton>
        </div>
        <div class="" ref="gpsLocation">
          Your GPS position is: {{ state.longitude }},
          {{ state.latitude }}
        </div>
      </div>
    </div>

    <div class="grid grid-flow-row">
      <div id="camera" class="web-camera-container">
        <div v-show="state.isCameraOpen && state.isLoading" class="camera-loading">
          <ul class="loader-circle">
            <li></li>
            <li></li>
            <li></li>
          </ul>
        </div>
        <div class="shadow w-full h-96 justify-items-center" v-show="!state.isLoading" >
          <video :width="400" :height="600" ref="camera" autoplay></video>
          <canvas  class="invisible absolute" id="photoTaken" ref="canvas" :width="400" :height="600" style="z-index: 0"></canvas>
        </div>

        <div v-if="state.isCameraOpen && !state.isLoading" class="grid justify-items-center">
          <button variant="success"  @click="takePhoto">
            <img src="https://img.icons8.com/material-outlined/50/000000/camera--v2.png">
          </button>
        </div>
      </div>
    </div>

    <div class="columns-1">
      <div v-if="state.pictureList.length >0 && !state.isLoading" class="w-full justify-center">
        <ImageCarousel :pictureList="state.pictureList" />
      </div>
    </div>


  </div>

</template>