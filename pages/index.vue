<template>
  <div class="container mx-auto p-5">
    <div class="grid">
      <div class="grid card bg-slate-500 dark:bg-zinc-700 p-5">
        <div class="card sm:w-96 mx-auto bg-base-100 shadow-xl">
          <video src="" id="local_video" autoplay width="100%"></video>
          <h1 class="text-center p-2" id="name"></h1>
          <div class="card-body">
            <div class="card-actions justify-end">
              <button class="btn btn-primary" id="webcambtn">
                Check Web Cam
              </button>
              <button id="get_link" class="btn btn-primary">Get Link</button>
            </div>
            <div class="card-action justify-between flex">
              <input
                type="text"
                id="input_link"
                class="input input-bordered w-full mr-2"
              /><button id="call_btn" class="btn btn-primary">Call</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "IndexPage",
  mounted() {
    const myvideo = document.getElementById("local_video");
    const webcambtn = document.getElementById("webcambtn");
    const get_link = document.getElementById("get_link");
    const call_btn = document.getElementById("call_btn");
    let room_id = null;

    webcambtn.onclick = async () => {
      myvideo.muted = true;
      navigator.mediaDevices
        .getUserMedia({
          video: true,
          audio: true,
        })
        .then((stream) => {
          myvideo.srcObject = stream;
        });
    };

    get_link.onclick = async () => {
      get_link.classList.add("loading")
      this.$axios.$post(process.env.SOCKET_URL_PORT + "/get_uuid").then((s) => {
        document.getElementById("input_link").value = s;
        room_id = s;
        get_link.classList.remove("loading")
      });
    };

    call_btn.onclick = async () => {
      window.location.href = "/room/" + room_id;
    };
  },
};
</script>
