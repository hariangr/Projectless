<template>
  <div id="wrapper">
    <img id="logo" src="~@/assets/logo.png" alt="electron-vue" />

    <div>
      <button @click="enableServer">Enable Server</button>
      <button @click="disableServer">Disable Server</button>
      <span v-if="this.webSocketServer != null">Koneksi {{ this.clientCount }}</span>

      <form class="slidecontainer">
        <input type="range" min="1" max="100" v-model="streamQuality" class="slider" id="myRange" />
        <input type="checkbox" name="ck" id="ck1" v-model="enableCompression" />

        <p v-if="this.enableCompression">Kompresi to source ratio {{latestCompressionRate}}</p>

        <h3>Kualitas {{streamQuality}}</h3>
      </form>
    </div>

    <main>
      <!-- <video id="videox" style="height: 100%;"></video> -->
      <canvas id="canvas" width="1280" height="720"></canvas>
    </main>
  </div>
</template>

<script>
import SystemInformation from "./LandingPage/SystemInformation";
import { desktopCapturer } from "electron";
import * as lzString from "./lz-string";

const WebSocket = require("ws");
import * as Express from "express";

export default {
  name: "dashboard-page",
  components: { SystemInformation },
  data() {
    return {
      clientCount: 0,
      webSocketServer: null,
      serverEnabled: false,
      expressServer: null,
      streamQuality: 30,
      enableCompression: true,
      latestCompressionRate: 0
    };
  },
  methods: {
    updateClientCount: function() {
      this.clientCount = this.webSocketServer.clients.size;
    },
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    enableExpress() {
      const express = require("express");
      this.expressServer = express();
      const port = 4000;

      this.expressServer.get("/", (req, res) => {
        // res.send("Hello World!");
        res.sendFile("view.html", { root: __dirname });
      });

      this.expressServer.listen(port, () =>
        console.log(`Example app listening on port ${port}!`)
      );
    },
    enableServer() {
      this.enableExpress();
      this.enableWebsocket();
      this.enableCamera();
    },
    disableServer() {
      this.expressServer = null;
      this.serverEnabled = false;
      this.webSocketServer = null;
    },
    enableCamera() {
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
    },
    handleStream(stream) {
      // const video = document.querySelector("#videox");
      const video = document.createElement("video");
      video.srcObject = stream;
      video.onloadedmetadata = e => video.play();

      console.log(stream);
      var options = { mimeType: "video/webm; codecs=vp9" };
      var mediaRecorder = new MediaRecorder(stream);

      // const canvas = document.createElement("canvas");
      const canvas = document.querySelector("#canvas");
      canvas.height = 720;
      canvas.width = 1280;
      const ctx = canvas.getContext("2d");

      setInterval(() => {
        console.log(this.streamQuality / 100);

        ctx.drawImage(video, 0, 0, 1280, 720);
        var base64Str = canvas.toDataURL(
          "image/jpeg",
          this.streamQuality / 100
        );
        this.sendImage(base64Str);
      }, 1000 / 24);
    },
    sendImage(message) {
      if (this.enableCompression) {
        let compressed = lzString.compressToUTF16(message);

        this.latestCompressionRate = (compressed.length / message.length) * 100;

        this.webSocketServer.clients.forEach(client => {
          client.send(
            JSON.stringify({ event: "imgbuffcompressed", data: compressed })
          );
        });
      } else {
        this.webSocketServer.clients.forEach(client => {
          client.send(JSON.stringify({ event: "imgbuff", data: message }));
        });
      }
    },
    handleMessage(ws, message) {
      console.log("received: %s", message);
      let _understood = JSON.parse(message);
      let event = _understood.event;
      let payload = _understood.data;

      switch (event) {
        case "cheese":
          this.handleCheese(ws, payload);
          break;
        case "save":
          this.handleSave(ws, payload);
          break;
        case "ping":
          this.handlePing(ws, payload);
          break;
        case "new":
          this.handleNew(ws, payload);
          break;
        default:
          break;
      }

      ws.send("somethingx");
    },
    handleConnection(ws) {
      this.updateClientCount();

      ws.on(
        "close",
        function() {
          console.log("closed");
          this.updateClientCount();
        }.bind(this)
      );

      ws.on("disconnect", function() {
        console.log("user disconnected");
      });

      ws.on("message", msg => this.handleMessage(ws, msg));
    },
    enableWebsocket() {
      console.log("ws");

      this.serverEnabled = true;
      this.webSocketServer = new WebSocket.Server({ port: 3000 });

      this.webSocketServer.on("connection", this.handleConnection);
    }
  }
};
</script>

<style>
img {
  object-fit: contain;
}
</style>
