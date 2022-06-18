<template>
  <div class="container mx-auto p-5">
    <div class="sm:grid grid-cols-3">
      <div
        class="
          grid grid-cols-1
          gap-5
          col-span-1
          bg-slate-400
          dark:bg-zinc-700
          p-5
        "
      >
        <div class="card bg-base-100 shadow-xl">
          <video src="" id="local_video" autoplay width="100%"></video>
          <h1 class="text-center p-2" id="name"></h1>
          <div class="card-actions justify-center">
            <div class="flex items-center mb-3">
              <label for="" class="mr-2">CAM</label>
              <input type="checkbox" class="toggle" id="cam_toggle" checked />
            </div>
            <div class="flex items-center">
              <label for="" class="mr-2">MIC</label>
              <input type="checkbox" class="toggle" id="mic_toggle" checked />
            </div>
          </div>
        </div>
      </div>
      <div
        class="
          bg-slate-400
          dark:bg-zinc-700
          p-5
          col-span-2
          grid grid-cols-1
          sm:grid-cols-2
        "
      >
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
    const cam_toggle = document.getElementById("cam_toggle");
    const mic_toggle = document.getElementById("mic_toggle");

    myvideo.muted = true;

    let myVideo = null;

    var peer = new Peer(undefined, {
      path: "/peerjs",
      host: process.env.SOCKET_URL,
      // port: 4000,
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
        myVideo = stream;

        peer.on("call", (call) => {
          call.answer(stream);
          const video = document.createElement("video");
          call.on("stream", (userVideoStream) => {
            addVideoStream(video, userVideoStream);
          });
        });

        socket.on("user-connected", (userId) => {
          document.getElementById("show_user_id").innerHTML =
            "user connected : " + userId;
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
      video.classList.add("shadow-xl");
      video.addEventListener("loadedmetadata", () => {
        video.play();
      });
      videoGrid.append(video);
    }

    setInterval(() => {
      document.getElementById("show_user_id").innerHTML = "";
    }, 10000);

    cam_toggle.addEventListener("click", () => {
      const enabled = myVideo.getVideoTracks()[0].enabled;
      if (enabled) {
        myVideo.getVideoTracks()[0].enabled = false;
      } else {
        myVideo.getVideoTracks()[0].enabled = true;
      }
    });

    mic_toggle.addEventListener("click", () => {
      const enabled = myVideo.getAudioTracks()[0].enabled;
      if (enabled) {
        myVideo.getAudioTracks()[0].enabled = false;
      } else {
        myVideo.getAudioTracks()[0].enabled = true;
      }
    });
  },
};
</script>

<style>
</style>