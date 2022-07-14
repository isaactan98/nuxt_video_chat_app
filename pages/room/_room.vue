<template>
  <div class="mx-auto p-5">
    <div class="sm:grid grid-cols-4 card shadow-xl">
      <div class="bg-slate-400 dark:bg-zinc-700 p-5">
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
          <div class="card-actions justify-center py-2">
            <button class="btn btn-error btn-sm" id="hangup_btn">X</button>
          </div>
        </div>
      </div>
      <div class="bg-slate-400 dark:bg-zinc-700 p-5 col-span-3">
        <div id="vide_grid" class="inline-flex"></div>
      </div>
      <div id="show_user_id" class="p-2"></div>
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

    // const user = prompt("Enter your name");
    const user = localStorage.getItem("name") ?? "User";

    document.getElementById("name").innerHTML = user;

    const socket = io(process.env.SOCKET_URL_PORT);

    const myvideo = document.getElementById("local_video");
    const videoGrid = document.getElementById("vide_grid");
    const cam_toggle = document.getElementById("cam_toggle");
    const mic_toggle = document.getElementById("mic_toggle");
    const hangup_btn = document.getElementById("hangup_btn");

    myvideo.muted = true;

    let mystream = null;
    let peer_id = null;

    var peer = new Peer(undefined, {
      path: "/peerjs",
      host: process.env.SOCKET_URL,
      // port: 4000,
    });

    const peers = {};
    navigator.mediaDevices
      .getUserMedia({
        video: true,
        audio: true,
      })
      .then((stream) => {
        myvideo.srcObject = stream;
        mystream = stream;

        peer.on("call", (call) => {
          call.answer(stream);

          // console.log(call);

          const video = document.createElement("video");
          const div = document.createElement("div");
          const span = document.createElement("span");
          span.innerHTML = call.metadata.user_name;
          div.setAttribute("data-id", call.metadata.user_id);

          call.on("stream", (userVideoStream) => {
            addVideoStream(video, userVideoStream, div, span);
          });
        });

        socket.on("user-connected", (userId, userName) => {
          document.getElementById("show_user_id").innerHTML =
            "user connected : " + userName;
          document
            .getElementById("show_user_id")
            .classList.add("text-green-500");
          document
            .getElementById("show_user_id")
            .classList.remove("text-red-500");
          connectToNewUser(userId, stream, userName);
          update_show_user();
        });

        socket.on("user-disconnected", (user_id, userName) => {
          document.getElementById("show_user_id").innerHTML =
            "user disconnected : " + userName;
          document
            .getElementById("show_user_id")
            .classList.remove("text-green-500");
          document.getElementById("show_user_id").classList.add("text-red-500");
          update_show_user();
        });
      });

    socket.on("user-disconnected", (userId) => {
      if (peers[userId]) peers[userId].close();
    });

    peer.on("open", (id) => {
      peer_id = id;
      socket.emit("join-room", this.param, id, user);
    });

    function connectToNewUser(userId, stream, userName) {
      let options = {
        metadata: {
          user_id: peer_id,
          user_name: user,
        },
      };
      const call = peer.call(userId, stream, options);
      const video = document.createElement("video");
      const div = document.createElement("div");
      const span = document.createElement("span");

      span.innerHTML = userName;
      div.setAttribute("id", userId);

      call.on("stream", (userVideoStream) => {
        addVideoStream(video, userVideoStream, div, span);
      });
      call.on("close", () => {
        document.getElementById(userId).remove();
      });

      peers[userId] = call;
    }

    function addVideoStream(video, stream, div, span) {
      video.srcObject = stream;

      span.classList.add("badge");
      span.classList.add("glass");
      span.classList.add("text-zinc-800");
      span.classList.add("mx-auto");
      span.classList.add("absolute");
      span.classList.add("bottom-0");
      span.classList.add("left-[25%]");
      span.classList.add("w-1/2");

      div.classList.add("card");
      div.classList.add("m-2");
      div.classList.add("shadow-xl");

      video.addEventListener("loadedmetadata", () => {
        video.play();
      });

      div.append(video, span);

      videoGrid.append(div);
    }

    function update_show_user() {
      setTimeout(() => {
        document.getElementById("show_user_id").innerHTML = "";
      }, 10000);
    }

    cam_toggle.addEventListener("click", () => {
      const enabled = mystream.getVideoTracks()[0].enabled;

      if (enabled) {
        mystream.getVideoTracks()[0].enabled = false;
      } else {
        mystream.getVideoTracks()[0].enabled = true;
      }
    });

    mic_toggle.addEventListener("click", () => {
      const enabled = mystream.getAudioTracks()[0].enabled;
      if (enabled) {
        mystream.getAudioTracks()[0].enabled = false;
      } else {
        mystream.getAudioTracks()[0].enabled = true;
      }
    });

    hangup_btn.addEventListener("click", function () {
      peer.on("close", (cnn) => {
        cnn.close();
      });
      window.location.href = "/";
    });
  },
};
</script>

<style>
</style>