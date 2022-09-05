<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <input type="file" ref="preview" @change="uploadFile" />
  <button v-on:click="hoge">upload</button>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {},
  methods: {
    async hoge() {
      var ocrText = "";
      const sleep = (waitTime) =>
        new Promise((resolve) => setTimeout(resolve, waitTime));
      const COMPUTER_VISION_API_ENDPOINT_URL =
        "https://japaneast.api.cognitive.microsoft.com/vision/v3.2/read/analyze";
      const TEXT_ANALYTICS_API_ENDPOINT_URL =
        "https://omoikane-analytics.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01";

      var config = {
        url: COMPUTER_VISION_API_ENDPOINT_URL,
        method: "post",
        headers: {
          "Content-type": "application/octet-stream",
          "Ocp-Apim-Subscription-Key": "82b9da35112c4f89941a116f68b14fd1",
        },
        data: this.$refs.preview.files[0],
      };

      // ocr処理
      await axios.request(config).then(async function (response) {
        config = {
          url: response.headers["operation-location"],
          method: "get",
          headers: {
            "Ocp-Apim-Subscription-Key": "82b9da35112c4f89941a116f68b14fd1",
          },
        };
        //2秒待つ
        await sleep(2000);

        // ocrデータの取得
        axios.request(config).then(async function (response) {
          var result = response.data.analyzeResult;
          // データの整形
          for (let i = 0; i < result.readResults.length; i++) {
            const resultRegionsLines = result.readResults[i].lines;
            for (let j = 0; j < resultRegionsLines.length; j++) {
              const resultRegionsLinesWords = resultRegionsLines[j].words;
              var currentLineTexts = "";
              for (let k = 0; k < resultRegionsLinesWords.length; k++) {
                currentLineTexts += resultRegionsLinesWords[k].text;
                ocrText += resultRegionsLinesWords[k].text;
              }
              console.log(currentLineTexts);
            }
          }
          // キーフレーズ取得
          config = {
            url: TEXT_ANALYTICS_API_ENDPOINT_URL,
            method: "post",
            headers: {
              "Content-type": "application/json",
              "Ocp-Apim-Subscription-Key": "fdd507dc09a44bf6a50dc6b4f60982dd",
            },
            data: {
              kind: "KeyPhraseExtraction",
              analysisInput: {
                documents: [
                  {
                    id: "1",
                    text: ocrText,
                    language: "ja",
                  },
                ],
              },
            },
          };
          response = await axios.request(config);
          console.log(response.data.results.documents[0].keyPhrases);
        });
      });
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
