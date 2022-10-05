<script setup>
import {ref, onMounted, reactive} from 'vue';
import ImageCarousel from "@/Components/ImageCarousel.vue";

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
    video: true
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
  context.drawImage(camera.value, 0, 0, 450, 337.5);
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
  <div ref="gpsLocation">
    Your GPS position is: {{ state.longitude }},
    {{ state.latitude }}
  </div>
  <div id="camera" class="web-camera-container">
    <div class="camera-button">
      <button type="button" class="button is-rounded"
              :class="{ 'is-primary' : !state.isCameraOpen, 'is-danger' : state.isCameraOpen}" @click="toggleCamera">
        <span v-if="!state.isCameraOpen">Open Camera</span>
        <span v-else>Close Camera</span>
      </button>
    </div>

    <div v-show="state.isCameraOpen && state.isLoading" class="camera-loading">
      <ul class="loader-circle">
        <li></li>
        <li></li>
        <li></li>
      </ul>
    </div>

    <div v-if="state.isCameraOpen" v-show="!state.isLoading" class="camera-box"
         :class="{ 'flash' : state.isShotPhoto }">

      <div class="camera-shutter" :class="{'flash' : state.isShotPhoto}"></div>

      <video ref="camera" width="450" height="337.5" autoplay style="z-index: 100"></video>

      <canvas hidden id="photoTaken" ref="canvas" :width="450" :height="337.5" style="z-index: 0"></canvas>
    </div>

    <div v-if="state.isCameraOpen && !state.isLoading" class="camera-shoot">
      <button type="button" class="button" @click="takePhoto">
        <img src="https://img.icons8.com/material-outlined/50/000000/camera--v2.png">
      </button>
    </div>
  </div>

  <ImageCarousel :pictureList="state.pictureList" />
</template>