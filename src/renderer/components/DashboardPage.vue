<template>
  <div id="wrapper">
    <img id="logo" src="~@/assets/logo.png" alt="electron-vue" />
    <div>
      <button @click="enableServer">Enable Server</button>
    </div>

    <main>
      <video id="videox" style="width: 100%; height: 100%;"></video>

      <div v-if="webSocketServer != null">
        <ul id="aaa">
          <li v-for="(con, i) in webSocketServer.clients.entries" :key="i">hi</li>
        </ul>
      </div>
    </main>
  </div>
</template>

<script>
import SystemInformation from "./LandingPage/SystemInformation";
import { desktopCapturer } from "electron";
const WebSocket = require("ws");
import * as Express from "express";

export default {
  name: "dashboard-page",
  components: { SystemInformation },
  data() {
    return {
      connections: [],
      webSocketServer: null,
      serverEnabled: false
    };
  },
  methods: {
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    enableExpress() {
      const express = require("express");
      const app = express();
      const port = 4000;

      app.get("/", (req, res) => {
        // res.send("Hello World!");
        res.sendFile("view.html", { root: __dirname });
      });

      app.listen(port, () =>
        console.log(`Example app listening on port ${port}!`)
      );
    },
    enableServer() {
      this.enableExpress();
      this.enableWebsocket();
      this.enableCamera();
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
      console.log("hal");

      const video = document.querySelector("#videox");
      video.srcObject = stream;
      video.onloadedmetadata = e => video.play();

      const judulvid = document.querySelector("#judulvid");

      console.log(stream);
      var options = { mimeType: "video/webm; codecs=vp9" };
      var mediaRecorder = new MediaRecorder(stream);

      const canvas = document.createElement("canvas");
      canvas.height = 720;
      canvas.width = 1280;
      const ctx = canvas.getContext("2d");

      setInterval(() => {
        ctx.drawImage(video, 0, 0, 1280, 720);
        var base64Str = canvas.toDataURL("image/jpeg", 0.5);
        this.sendImage(base64Str);
      }, 1000 / 24);
    },
    sendImage(message) {
      this.webSocketServer.clients.forEach(client => {
        client.send(JSON.stringify({ event: "imgbuff", data: message }));
      });
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
      console.log("a user connected");

      console.log(this.webSocketServer.clients);

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
    },
    disableWebsocket() {
      this.serverEnabled = false;
      this.webSocketServer = null;
    }
  }
};
</script>

<style>
img {
  object-fit: contain;
}
</style>
