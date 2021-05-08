<template>
  <div>
    <div class="centerColumn">
      <img id="hdrImg" src="uaemex2.png" alt="" />
      <div id="header">
        <h2>Sistema de verificación biométrica</h2>
      </div>
      <table id="cameraInput">
        <tr>
          <td>
            <video
              id="webcam"
              autoplay
              playsinline
              width="320"
              height="240"
            ></video>
          </td>
          <td id="canvasPlaceholder" v-if="!this.lastPicture">
            <img id="canvasPlaceholder" src="silueta.jpg" alt="no encontrada" />
          </td>
          <td v-show="this.lastPicture">
            <canvas
              id="canvas"
              class="d-none"
              width="320"
              height="240"
            ></canvas>
          </td>
        </tr>
        <tr>
          <td colspan="100%">
            <div class="belowFrames">
              <button class="captureButton" v-on:click="snapPicture">
                Capturar
              </button>
            </div>
          </td>
        </tr>
      </table>

      <audio
        id="snapSound"
        src="audio/snap.wav"
        preload="auto"
        visibility:hidden
      ></audio>
      <div class="input">
        <table id="inputTable">
          <tr>
            <td>Número de cuenta</td>
            <td>
              <input name="numero-cuenta" type="text" />
            </td>
          </tr>
          <tr>
            <td>Contraseña</td>
            <td><input type="password" /></td>
          </tr>
          <tr>
            <td colspan="100%">
              <div class="belowFrames"><button>Enviar</button></div>
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line
import Webcam from "webcam-easy";

export default {
  name: "MainSplash",
  data() {
    return {
      webcam: null,
      lastPicture: null,
      numCuenta: null,
      password: null
    };
  },
  mounted: function() {
    const webcamElement = document.getElementById("webcam");
    const canvasElement = document.getElementById("canvas");
    const snapSoundElement = document.getElementById("snapSound");
    this.webcam = new Webcam(
      webcamElement,
      "user",
      canvasElement,
      snapSoundElement
    );

    this.webcam
      .start()
      .then(result => {
        console.log("webcam started");
        console.log(result);
      })
      .catch(err => {
        console.log("Error al inicializar la cámara");
        console.error(err);
      });
  },
  methods: {
    snapPicture: function() {
      this.lastPicture = this.webcam.snap();
    }
  }
};
</script>

<style>
.centerColumn {
  margin-left: auto;
  margin-right: auto;
  width: 50vw;
  display: block;
}

#cameraInput {
  /* border: 1px solid green; */
  margin-bottom: 2vw;
}

/* td { */
/* border: 1px solid; */
/* display: table-cell; */
/* } */

table#cameraInput td {
  margin-bottom: 1em;
  margin-top: 1em;
}

table#inputTable td {
  padding-top: 0.5em;
  padding-bottom: 0.5em;
  padding-left: 0.5em;
  padding-right: 0.5em;
}

#inputTable {
  margin-left: auto;
  margin-right: auto;
  border: 1px solid grey;
}

#canvasPlaceholder {
  object-fit: cover;
  width: 320px;
  height: 240px;
}

.belowFrames {
  margin-left: auto;
  margin-right: auto;
  margin-top: 2vw;
}

#hdrImg {
  max-width: 70%;
  height: auto;
}

#header {
  color: #292929;
  background-color: #cbcbcb;
}

.captureButton {
  background-color: #ff4c4c;
  color: whitesmoke;
  padding: 5px;
  font-size: 15px;
  border: none;
  border-radius: 5px;
  font-style: bold;
}
</style>
