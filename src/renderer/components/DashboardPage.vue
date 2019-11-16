<template>
  <div id="wrapper">
    <img id="logo" src="~@/assets/logo.png" alt="electron-vue" />
    <main>
      <div class="left-side">
        <span class="title">Welcome to your old project!</span>
        <system-information></system-information>
      </div>

      <button @click="capturerEnable">Tekan</button>
      <button @click="enableServer">Enable Server</button>
      <video id="judulvid" width="320" height="240" controls></video>
      <video id="videox" width="320" height="240" controls></video>
    </main>
  </div>
</template>

<script>
import SystemInformation from "./LandingPage/SystemInformation";
import { desktopCapturer } from "electron";
import * as Express from "express";

export default {
  name: "dashboard-page",
  components: { SystemInformation },
  methods: {
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    enableServer() {
      console.log("Ape kaden");

      const express = require("express");
      const app = express();
      const port = 3099;

      app.get("/", (req, res) => res.send("Hello World!"));

      app.listen(port, () =>
        console.log(`Example app listening on port ${port}!`)
      );
    },
    handleStream(stream) {
      console.log("hal");

      const video = document.querySelector("#videox");
      video.srcObject = stream;
      video.onloadedmetadata = e => video.play();

      const judulvid = document.querySelector("#judulvid");
      //   judulvid.srcObject = stream;
      //   judulvid.onloadedmetadata = e => judulvid.play();

      console.log(stream);
      var options = { mimeType: "video/webm; codecs=vp9" };
      var mediaRecorder = new MediaRecorder(stream);

      mediaRecorder.ondataavailable = function(event) {
        if (event.data.size > 0) {
          console.log(event);
          judulvid.src = window.URL.createObjectURL(event.data);
        //   video.onloadedmetadata = e => video.play();
          video.play();
        } else {
          // ...
        }
      };

      mediaRecorder.start(2000);

      //   setTimeout(() => {
      //     mediaRecorder.stop();
      //   }, 9000);
      //   //   recorder.start();
    },
    handleError(e) {
      console.log(e);
    },
    capturerEnable() {
      desktopCapturer.getSources(
        { types: ["window", "screen"] },
        (err, sources) => {
          sources.forEach(async element => {
            console.log(element);

            if (element.name == "Entire screen") {
              try {
                const stream = await navigator.mediaDevices.getUserMedia({
                  audio: false,
                  video: {
                    mandatory: {
                      chromeMediaSource: "desktop",
                      // chromeMediaSourceId: source.id,
                      minWidth: 1280,
                      maxWidth: 1280,
                      minHeight: 720,
                      maxHeight: 720
                    }
                  }
                });

                this.handleStream(stream);
              } catch (e) {
                this.handleError(e);
              }
            }
          });
        }
      );
    }
  }
};
</script>

<style>
</style>
