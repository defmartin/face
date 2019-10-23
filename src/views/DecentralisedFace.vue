<template class="bg">
  <div class="hello">
    <div class="canvas">
      <div v-for="item in boardArray" v-bind:key="item.id">
        <Pixel class="small pixel" v-bind:color="item.color" />
      </div>
    </div>
    <!-- end .canvas -->
  </div>
</template> 


<script>
import {
  radixUniverse,
  RadixUniverse,
  radixTokenManager,
  RadixKeyStore,
  RadixSimpleIdentity,
  RadixIdentityManager,
  RadixAccount,
  RadixTransactionBuilder,
  RRI
} from "radixdlt";

import Pixel from "../components/Pixel.vue";

export default {
  name: "FaceComp",
  props: {
    msg: String
  },
  components: {
    Pixel
  },
  data() {
    return {
      boardArray: [],
      boardSize: 2500,
      account: null,
      pixels: []
    };
  },
  watch: {
    pixels: function(val) {
      this.updateBoard();
    }
  },
  methods: {
    genBoard() {
      for (let index = 0; index < this.boardSize; index++) {
        this.boardArray.push({ id: index, color: "#000" });
      }
    },
    updateBoard() {
      this.pixels.map(function(item) {
        let newColor = this.fullColorHex(item[1], item[2], item[3]);
        this.boardArray[item[0]]["color"] = newColor;
      }, this);
    },
    rgbToHex(rgb) {
      var hex = Number(rgb).toString(16);
      if (hex.length < 2) {
        hex = "0" + hex;
      }
      return hex;
    },
    fullColorHex(r, g, b) {
      var red = this.rgbToHex(r);
      var green = this.rgbToHex(g);
      var blue = this.rgbToHex(b);
      return "#" + red + green + blue;
    },
    getTheseJuicyPixels() {
      this.account = RadixAccount.fromAddress(
        "9humTFFUAqhtFLNUpG4CoNB9ouRJPpzAtt9x4PNeqcLTsaTdbvp"
      );
      this.account.openNodeConnection();
      console.log("account:", this.account.getAddress());

      this.account.messagingSystem.getAllMessages().subscribe(data => {
        const dataJSON = JSON.parse(data.message.content);
        this.pixels.push(dataJSON);
        this.pixels = Array.from(new Set(this.pixels));

      });

      //   this.account.dataSystem.getApplicationData("dan").subscribe(data => {
      //     const dataJSON = JSON.parse(data.data.payload.data);
      //     this.pixels.push(dataJSON);
      //     this.pixels = Array.from(new Set(this.pixels));
      //     console.log(dataJSON[0])
      //   });
    }
  },
  created() {
    // Bootstrap the universe
    radixUniverse.bootstrap(RadixUniverse.BETANET_EMULATOR);
    this.genBoard();
    this.getTheseJuicyPixels();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.canvas {
  /* Perfectly square */
  width: 750px;
  height: 750px;
  padding-left: 0;
    padding-right: 0;
    margin-left: auto;
    margin-right: auto;
    display: block;
}
.bg {
  background-color: #000;
}
.pixel {
  /* We'll need 64 total pixels in our HTML */
  width: 15px;
  height: 15px;
  float: left;
  /* box-shadow: 0px 0px 1px #42b983; */
  background-color: #000;
}

.column {
  float: left;
  width: 50%;
}

/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}

html{
    background-color: #000;
}
</style>
