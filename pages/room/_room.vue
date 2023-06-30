<template>
  <div class="mx-auto p-5 relative w-full">
    <link rel="stylesheet"
      href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@48,400,0,0" />
    <div id="show_user_id" class="p-2"></div>
    <div class="flex flex-wrap shadow-xl mb-10" id="show_grid">
      <div class="p-5 flex-grow">
        <div class="card bg-base-100 shadow-xl">
          <video src="" id="local_video" autoplay width="100%" playsinline></video>
          <h1 class="text-center p-2" id="name"></h1>
        </div>
      </div>
      <div id="vide_grid"></div>
    </div>
    <!-- Footer Action -->
    <div class="fixed bottom-0 w-full -ml-5">
      <div class="flex justify-center items-center p-5 bg-white dark:bg-zinc-800">
        <div class="flex items-center mr-3">
          <label for="" class="mr-2">CAM</label>
          <input type="checkbox" class="toggle" id="cam_toggle" checked />
        </div>
        <div class="flex items-center mr-3">
          <label for="" class="mr-2">MIC</label>
          <input type="checkbox" class="toggle" id="mic_toggle" checked />
        </div>
        <button class="btn btn-sm rounded-full bg-red-500" id="hangup_btn">
          <span class="material-symbols-outlined text-white"> call_end </span>
        </button>
      </div>
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
    const show_grid = document.getElementById("show_grid");
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
          const outerdiv = document.createElement("div");
          const innerdiv = document.createElement("div");
          const span = document.createElement("span");
          span.innerHTML = call.metadata.user_name;
          outerdiv.setAttribute("id", call.metadata.user_id);

          call.on("stream", (userVideoStream) => {
            addVideoStream(video, userVideoStream, outerdiv, innerdiv, span);
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
      const outerdiv = document.createElement("div");
      const innerdiv = document.createElement("div");
      const span = document.createElement("span");

      span.innerHTML = userName;
      outerdiv.setAttribute("id", userId);

      call.on("stream", (userVideoStream) => {
        addVideoStream(video, userVideoStream, outerdiv, innerdiv, span);
      });
      call.on("close", () => {
        document.getElementById(userId).remove();
      });

      peers[userId] = call;
    }

    function addVideoStream(video, stream, outerdiv, innerdiv, span) {
      video.srcObject = stream;
      video.setAttribute("playsinline", true);

      span.classList.add("text-center");
      span.classList.add("p-2");

      innerdiv.classList.add("card");
      innerdiv.classList.add("shadow-xl");

      video.addEventListener("loadedmetadata", () => {
        video.play();
      });

      innerdiv.append(video, span)

      outerdiv.classList.add("p-5");
      outerdiv.classList.add("flex-grow");

      outerdiv.append(innerdiv);

      show_grid.insertBefore(outerdiv, videoGrid.children[0]);
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
@media screen and (max-width: 960) {
  video {
    max-width: 360px;
  }
}
</style>