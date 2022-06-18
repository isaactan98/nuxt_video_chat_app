<template>
  <div class="container mx-auto p-5">
    <div class="grid grid-cols-3">
      <div class="grid grid-cols-1 gap-5 col-span-1 bg-white p-5">
        <div class="card bg-base-100 shadow-xl">
          <video src="" id="local_video" autoplay width="100%"></video>
          <h1 class="text-center p-2" id="name"></h1>
          <div class="card-body">
            <div class="card-actions justify-end">
              <button class="btn btn-primary" id="webcambtn">
                Check Web Cam
              </button>
              <button id="get_link" class="btn btn-primary">Get Link</button>
            </div>
          </div>
        </div>
      </div>
      <div class="bg-slate-400 p-5 col-span-2 grid grid-cols-2">
        <div id="vide_grid"></div>
      </div>
      <div id="show_user_id"></div>
    </div>
  </div>
</template>

<script>
import { io } from "socket.io-client";

export default {
  data() {
    return {
      param: null,
    };
  },
  head() {
    return {
      title: "Room " + this.$route.params.room,
    };
  },
  mounted() {
    this.param = this.$route.params.room;

    if (this.param == null) {
      window.location.href = "/";
    }

    const socket = io(process.env.SOCKET_URL_PORT);

    const webcambtn = document.getElementById("webcambtn");
    const myvideo = document.getElementById("local_video");
    const videoGrid = document.getElementById("vide_grid");
    const get_link_btn = document.getElementById("get_link");
    const input_link = document.getElementById("input_link");
    const call_btn = document.getElementById("call_btn");

    myvideo.muted = true;

    var peer = new Peer(undefined, {
      path: "/peerjs",
      host: process.env.SOCKET_URL,
    });

    myvideo.muted = true;
    const peers = {};
    navigator.mediaDevices
      .getUserMedia({
        video: true,
        audio: true,
      })
      .then((stream) => {
        myvideo.srcObject = stream;

        peer.on("call", (call) => {
          call.answer(stream);
          const video = document.createElement("video");
          call.on("stream", (userVideoStream) => {
            addVideoStream(video, userVideoStream);
          });
        });

        socket.on("user-connected", (userId) => {
          document.getElementById("show_user_id").innerHTML = userId;
          connectToNewUser(userId, stream);
        });
      });

    socket.on("user-disconnected", (userId) => {
      if (peers[userId]) peers[userId].close();
    });

    peer.on("open", (id) => {
      socket.emit("join-room", this.param, id);
    });

    function connectToNewUser(userId, stream) {
      const call = peer.call(userId, stream);
      const video = document.createElement("video");
      video.setAttribute("id", userId);
      video.classList.add("card");
      call.on("stream", (userVideoStream) => {
        addVideoStream(video, userVideoStream);
      });
      call.on("close", () => {
        document.getElementById(userId).remove();
      });

      peers[userId] = call;
    }

    function addVideoStream(video, stream) {
      video.srcObject = stream;
      video.classList.add("card");
      video.addEventListener("loadedmetadata", () => {
        video.play();
      });
      videoGrid.append(video);
    }

    setInterval(() => {
      document.getElementById("show_user_id").innerHTML = "";
    }, 10000);
  },
};
</script>

<style>
</style>