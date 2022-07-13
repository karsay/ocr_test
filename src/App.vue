<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <input type="file" ref="preview" @change="uploadFile" />
  <button v-on:click="hoge">upload</button>
</template>

<script>
import axios from 'axios'

export default {
  name: "App",
  contentBuffer:{
    aaa:"hoge"
  },
  methods: {
    async hoge() {
      const COMPUTER_VISION_API_ENDPOINT_URL =
        "https://japaneast.api.cognitive.microsoft.com/vision/v3.0/ocr?language=unk&detectOrientation=true";

      const config = {
        url: COMPUTER_VISION_API_ENDPOINT_URL,
        method: "post",
        headers: {
          "Content-type": "application/octet-stream",
          "Ocp-Apim-Subscription-Key": "82b9da35112c4f89941a116f68b14fd1",
        },
        data: this.$refs.preview.files[0],
      };

      const responseAzure = await axios.request(config);
      var result = responseAzure.data
      for (let i=0; i<result.regions.length; i++){
            const resultRegionsLines = result.regions[i].lines;
            for (let j=0; j<resultRegionsLines.length; j++) {
                const resultRegionsLinesWords = resultRegionsLines[j].words;
                let currentLineTexts = "";
                for (let k=0; k<resultRegionsLinesWords.length; k++) {
                    currentLineTexts += resultRegionsLinesWords[k].text;
                }
                console.log(currentLineTexts);
            }
      }
      console.log("post OK");
    },
    uploadFile() {
      console.log(this.$refs.preview.files[0]);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
