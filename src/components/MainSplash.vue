<template>
  <div>
    <div class="centerColumn" v-show="this.state == states.AWAITING_INPUT">
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
              <input name="numero-cuenta" type="text" v-model="numCuenta" />
            </td>
          </tr>
          <tr>
            <td>Contraseña</td>
            <td><input type="password" v-model="password" /></td>
          </tr>
          <tr>
            <td colspan="100%">
              <div class="belowFrames">
                <button v-on:click="makeRequest">Enviar</button>
              </div>
            </td>
          </tr>
        </table>
      </div>
    </div>
    <div class="centerColumn" v-if="this.state == states.PROCESSING">
      <h1>Espere un momento por favor...</h1>
      <Circle2 class="centerColumn"></Circle2>
    </div>
    <div class="centerColumn" v-if="this.state == states.RECEIVED_RESPONSE">
      <div v-if="this.verified">
        <h1>Identidad validada exitosamente</h1>
        <img class="acceptanceImg centerColumn" src="ok.webp" alt="" />
        <div id="hash-frame">
          Bienvenido:
          <br />
          <div class="grey-frame">
            {{ primer_apellido }} {{ segundo_apellido }}, {{ nombres }}
          </div>
          <br />
          Firma sigital:
          <div class="grey-frame">{{ key }}</div>
        </div>
      </div>
      <div v-else>
        <h1>No fue posible validar su identidad</h1>
        <img class="acceptanceImg centerColumn" src="not_ok.webp" alt="" />
        <button v-on:click="reset">Intentar de nuevo</button>
      </div>
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line
import Webcam from "webcam-easy";
import axios from "axios";
import sha256 from "crypto-js/sha256";
import Base64 from "crypto-js/enc-base64";
import { Circle2 } from "vue-loading-spinner";

const states = {
  AWAITING_INPUT: "Esperando entrada",
  PROCESSING: "Procesando imágen",
  RECEIVED_RESPONSE: "Respuesta recibida"
};

export default {
  name: "MainSplash",
  components: {
    Circle2
  },
  data() {
    return {
      webcam: null,
      lastPicture: null,
      numCuenta: "",
      password: "",
      state: states.AWAITING_INPUT,
      states,
      verified: false,
      key: "",
      nombres: "",
      primer_apellido: "",
      segundo_apellido: ""
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
      })
      .catch(err => {
        console.log("Error al inicializar la cámara");
        console.error(err);
      });
  },
  methods: {
    snapPicture: function() {
      this.lastPicture = this.webcam.snap();
    },
    makeRequest: function() {
      this.state = states.PROCESSING;

      let formData = new FormData();
      let hash = sha256(this.password);

      axios
        .post("http://localhost:5000/api/validar", {
          num_cuenta: this.numCuenta,
          hash: hash.toString(),
          base64_encoded_photo: this.lastPicture
        })
        .then(res => {
          console.log(res.data);
          this.state = states.RECEIVED_RESPONSE;
          this.verified = res.data.valid;
          this.key = res.data.key;
          this.nombres = res.data.nombres;
          this.primer_apellido = res.data.primer_apellido;
          this.segundo_apellido = res.data.segundo_apellido;
        })
        .catch(err => {
          console.error("Error!");
          console.log(err.response);
        });
    },
    reset: function() {
      this.lastPicture = null;
      this.password = "";
      this.state = states.AWAITING_INPUT;
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

.acceptanceImg {
  width: 200px;
  height: auto;
}

.grey-frame {
  background: #cdcdcd;
}

#hash-frame {
  font-size: 30px;
  font-weight: bold;
}
</style>
