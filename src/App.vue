<script setup>
import { ref } from 'vue';
import { listener, startTablet } from './helper/SigWeb';

const sigCaptured = ref(null);

const handleSig = () => {};
const handleSigCapture = (sig) => {
  let image = new Image();
  image.src = 'data:image/png;base64,' + sig;
  sigCaptured.value = { image, sig };
};

listener();
</script>

<template>
  <header>
    <h1>Capture Signature</h1>
  </header>
  <main>
    <h3>Signature</h3>
    <canvas id="cnv" name="cnv" width="500" height="100" />
    <form action="#" name="FORM1">
      <button v-if="sigCaptured" @click="handleSig">Done</button>
      <button v-else @click="startTablet(handleSigCapture)">Sign</button>
      <h5>SigString:</h5>
      <textarea rows="20" name="sigString" />
      <h5>ImgData:</h5>
      <textarea rows="20" name="imgData" :value="sigCaptured?.image?.src" />
      <h5>Image:</h5>
      <img v-if="sigCaptured" :src="sigCaptured?.image" alt="Signature" />
    </form>
  </main>
</template>

<style scoped>
header {
  text-align: center;
}

main {
  padding: 2rem;
}

canvas {
  margin: 0.5rem 0;
  border: red 1px solid;
}

button {
  display: block;
}

textarea {
  width: 100%;
}
</style>
