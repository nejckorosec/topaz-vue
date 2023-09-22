<script setup>
import { ref } from 'vue';
import {
  IsSigWebInstalled,
  GetSigWebVersion,
  isSigWeb_1_7_0_0_Installed,
  isSigWeb_1_6_4_0_Installed,
  GetDaysUntilCertificateExpires,
  SetDisplayXSize,
  SetDisplayYSize,
  SetTabletState,
  SetJustifyMode,
  ClearTablet,
  NumberOfTabletPoints,
  SetSigCompressionMode,
  SetImageXSize,
  SetImageYSize,
  SetImagePenWidth,
  GetSigImageB64,
  Reset,
  GetSigString,
} from './helper/SigWeb';

const tmr = ref(null);
const resetIsSupported = ref(false);
const SigWeb_1_6_4_0_IsInstalled = ref(false);
const SigWeb_1_7_0_0_IsInstalled = ref(false);

const bioSigData = ref('');
const sigStringData = ref('');
const sigImageData = ref('');
const sigWebVersionNote = ref(null);
const daysUntilExp = ref(null);
const sigCaptured = ref(null);

const listener = () => {
  if (IsSigWebInstalled()) {
    let sigWebVer = '';
    try {
      sigWebVer = GetSigWebVersion();
    } catch (err) {
      console.log('Unable to get SigWeb Version: ' + err.message);
    }

    if (sigWebVer != '') {
      try {
        SigWeb_1_7_0_0_IsInstalled.value =
          isSigWeb_1_7_0_0_Installed(sigWebVer);
      } catch (err) {
        console.log(err.message);
      }
      //if SigWeb 1.7.0.0 is installed, then enable corresponding functionality
      if (SigWeb_1_7_0_0_IsInstalled.value) {
        resetIsSupported.value = true;
        try {
          const daysUntilCertExpires = GetDaysUntilCertificateExpires();
          daysUntilExp.value =
            'SigWeb Certificate expires in ' + daysUntilCertExpires + ' days.';
        } catch (err) {
          console.log(err.message);
        }
        sigWebVersionNote.value = 'SigWeb 1.7.0 installed';
      } else {
        try {
          SigWeb_1_6_4_0_IsInstalled.value =
            isSigWeb_1_6_4_0_Installed(sigWebVer);
          //if SigWeb 1.6.4.0 is installed, then enable corresponding functionality
        } catch (err) {
          console.log(err.message);
        }
        if (SigWeb_1_6_4_0_IsInstalled.value) {
          resetIsSupported.value = true;
          sigWebVersionNote.value =
            'SigWeb 1.6.4 is installed. Install the newer version of SigWeb from the following link: https://www.topazsystems.com/software/sigweb.exe';
        } else {
          sigWebVersionNote.value =
            'A newer version of SigWeb is available. Please uninstall the currently installed version of SigWeb and then install the new version of SigWeb from the following link: https://www.topazsystems.com/software/sigweb.exe';
        }
      }
    } else {
      //Older version of SigWeb installed that does not support retrieving the version of SigWeb (Version 1.6.0.2 and older)
      sigWebVersionNote.value =
        'A newer version of SigWeb is available. Please uninstall the currently installed version of SigWeb and then install the new version of SigWeb from the following link: https://www.topazsystems.com/software/sigweb.exe';
    }
  } else {
    alert(
      'Unable to communicate with SigWeb. Please confirm that SigWeb is installed and running on this PC.'
    );
  }
};
const onSign = () => {
  if (IsSigWebInstalled()) {
    const ctx = document.getElementById('cnv').getContext('2d');
    SetDisplayXSize(500);
    SetDisplayYSize(100);
    SetTabletState(0, tmr);
    SetJustifyMode(0);
    ClearTablet();
    if (tmr.value == null) {
      tmr.value = SetTabletState(1, ctx, 50);
    } else {
      SetTabletState(0, tmr.value);
      tmr.value = null;
      tmr.value = SetTabletState(1, ctx, 50);
    }
  } else {
    alert(
      'Unable to communicate with SigWeb. Please confirm that SigWeb is installed and running on this PC.'
    );
  }
};
const onClear = () => {
  ClearTablet();
};
const onDone = () => {
  if (NumberOfTabletPoints() == 0) {
    alert('Please sign before continuing');
  } else {
    SetTabletState(0, tmr);
    //RETURN TOPAZ-FORMAT SIGSTRING
    SetSigCompressionMode(1);
    bioSigData.value = GetSigString();
    sigStringData.value = GetSigString();
    //this returns the signature in Topaz's own format, with biometric information

    //RETURN BMP BYTE ARRAY CONVERTED TO BASE64 STRING
    SetImageXSize(500);
    SetImageYSize(100);
    SetImagePenWidth(5);
    GetSigImageB64(SigImageCallback);

    // set image
    const image = new Image();
    image.src = 'data:image/png;base64,' + sigImageData.value;
    sigCaptured.value = image;
  }
};
const SigImageCallback = (str) => {
  sigImageData.value = str;
};
const endDemo = () => {
  ClearTablet();
  SetTabletState(0, tmr.value);
};
const close = () => {
  if (resetIsSupported) {
    Reset();
  } else {
    endDemo();
  }
};

window.onbeforeunload = function (evt) {
  close();
  clearInterval(tmr.value);
  evt.preventDefault(); //For Firefox, needed for browser closure
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
    <!-- <h3>Signature Capture</h3> -->
    <!-- <canvas name="SigImg" id="SigImg" width="500" height="100"></canvas> -->
    <p>{{ sigWebVersionNote }}</p>
    <form action="#" @submit.prevent name="FORM1">
      <div class="actions">
        <button @click="onSign" type="button">Sign</button>
        <button @click="onClear" type="button">Clear</button>
        <button @click="onDone" type="button">Done</button>
      </div>
      <h5>SigString:</h5>
      <textarea name="sigStringData" rows="20" cols="50">
      {{ sigStringData }}
      </textarea>
      <h5>Base64 String::</h5>
      <textarea name="sigImageData" rows="20" cols="50">
      {{ sigImageData }}
      </textarea>
      <h5>Image:</h5>
      <img v-if="sigCaptured" :src="sigCaptured?.src" alt="Signature" />

      <input type="hidden" name="bioSigData" v-model="bioSigData" />
      <input type="hidden" name="sigImgData" v-model="sigImageData" />

      <p v-if="daysUntilExp">{{ daysUntilExp }}</p>
      <p id="SigWebVersion"></p>
      <p id="SigWebTabletJSVersion"></p>
      <p id="CertificateExpirationDate"></p>
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

.actions {
  display: flex;
  gap: 1rem;
}

button {
  display: block;
}

textarea {
  width: 100%;
}
</style>
