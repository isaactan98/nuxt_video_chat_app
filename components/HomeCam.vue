<template>
  <div class="grid card px-5 py-10 sm:p-20">
    <div class="
        card
        flex-col
        lg:flex-row
        mx-auto
        sm:p-5
        shadow-xl
        dark:bg-zinc-800
      ">
      <video src="" class="rounded-lg shadow-lg flex-1 max-w-sm" id="local_video" autoplay width="100%"
        playsinline></video>
      <div class="card-body justify-between lg:p-8">
        <h2 class="card-title">Welcome <span id="show_name"></span></h2>
        <div class="grid gap-5">
          <input type="text" class="input input-bordered" id="name" placeholder="Enter your name" autocomplete="off" />
          <div class="card-actions justify-between">
            <button class="btn btn-primary btn-sm w-full" id="webcambtn" @click="showCamera()">
              Check Cam
            </button>
            <button class="btn btn-primary btn-sm" @click="toggleCamera()">
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                stroke="currentColor" class="w-6 h-6">
                <path stroke-linecap="round" stroke-linejoin="round"
                  d="M19.5 12c0-1.232-.046-2.453-.138-3.662a4.006 4.006 0 00-3.7-3.7 48.678 48.678 0 00-7.324 0 4.006 4.006 0 00-3.7 3.7c-.017.22-.032.441-.046.662M19.5 12l3-3m-3 3l-3-3m-12 3c0 1.232.046 2.453.138 3.662a4.006 4.006 0 003.7 3.7 48.656 48.656 0 007.324 0 4.006 4.006 0 003.7-3.7c.017-.22.032-.441.046-.662M4.5 12l3 3m-3-3l-3 3" />
              </svg>
            </button>
            <button id="get_link" class="btn btn-primary btn-sm">
              Get Link
            </button>
          </div>
          <div class="card-action justify-between flex">
            <input type="text" id="input_link" class="input input-bordered w-full mr-2" autocomplete="off" /><button
              id="call_btn" class="btn btn-primary">Call</button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="recentRoom">
      <div class="card mt-5">
        <h2 class="card-title mb-2">Recent Room</h2>
        <div class="card-actions justify-between">
          <input type="text" class="input input-bordered w-full" v-model="recentRoom" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cameraFace: 'front',
      recentRoom: null
    };
  },
  mounted() {
    const webcambtn = document.getElementById("webcambtn");
    const get_link = document.getElementById("get_link");
    const call_btn = document.getElementById("call_btn");
    const get_name = document.getElementById("name");

    this.checkIfDateExpired();
    this.recentRoom = localStorage.getItem("recent-room");

    if (localStorage.getItem("name")) {
      document.getElementById("show_name").innerHTML =
        localStorage.getItem("name");

      get_name.value = localStorage.getItem("name");
    }

    get_name.addEventListener("keyup", function (e) {
      document.getElementById("show_name").innerHTML = get_name.value;
    });

    get_link.onclick = async () => {
      get_link.classList.add("loading");
      this.$axios.$post(process.env.SOCKET_URL_PORT + "/get_uuid").then((s) => {
        document.getElementById("input_link").value = s;
        localStorage.setItem("recent-room", s)
        localStorage.setItem("recent-date", new Date().getTime());
        this.recentRoom = null;
        get_link.classList.remove("loading");
      });
    };

    call_btn.onclick = async () => {
      localStorage.setItem(
        "name",
        get_name.value != null && get_name.value.length > 0
          ? get_name.value
          : "User"
      );

      window.location.href =
        "/room/" + document.getElementById("input_link").value;
    };
  },
  methods: {
    showCamera() {
      const myvideo = document.getElementById("local_video");
      myvideo.srcObject = null;
      let camera = this.cameraFace == 'rear' ? { facingMode: { exact: 'environment' } } : "user";
      myvideo.muted = true;
      navigator.mediaDevices
        .getUserMedia({
          video: camera,
          audio: true,
        })
        .then((stream) => {
          myvideo.srcObject = stream;
        }).catch((e) => {
          alert("Camera not found");
          this.cameraFace = 'front';
          localStorage.setItem("cameraFace", this.cameraFace);
          this.showCamera();
        })
    },
    toggleCamera() {
      this.cameraFace = this.cameraFace == 'rear' ? 'front' : 'rear';
      localStorage.setItem("cameraFace", this.cameraFace);
      this.showCamera();
    },
    checkIfDateExpired() {
      let recentDate = localStorage.getItem("recent-date");
      if (recentDate) {
        let date = new Date().getTime();
        let diff = date - recentDate;
        let minutes = Math.floor(diff / 60000);
        if (minutes > 60) {
          localStorage.removeItem("recent-room");
          localStorage.removeItem("recent-date");
        }
      }
    }
  },
};
</script>