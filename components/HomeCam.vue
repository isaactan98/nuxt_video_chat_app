<template>
  <div class="grid card px-5 py-10 sm:p-20">
    <div
      class="
        card
        flex-col
        lg:flex-row
        mx-auto
        sm:p-5
        shadow-xl
        dark:bg-zinc-800
      "
    >
      <video
        src=""
        class="rounded-lg shadow-md flex-1"
        id="local_video"
        autoplay
        width="100%"
      ></video>
      <div class="card-body flex-1">
        <div class="card-body justify-between p-0 lg:p-8">
          <h2 class="card-title">Welcome <span id="show_name"></span></h2>
          <div class="grid gap-5">
            <input
              type="text"
              class="input input-bordered"
              id="name"
              placeholder="Enter your name"
              autocomplete="off"
            />

            <div class="card-actions justify-end sm:justify-between">
              <button class="btn btn-primary btn-sm" id="webcambtn">
                Check Web Cam
              </button>
              <button id="get_link" class="btn btn-primary btn-sm">
                Get Link
              </button>
            </div>
            <div class="card-action justify-between flex">
              <input
                type="text"
                id="input_link"
                class="input input-bordered w-full mr-2"
                autocomplete="off"
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
  mounted() {
    const myvideo = document.getElementById("local_video");
    const webcambtn = document.getElementById("webcambtn");
    const get_link = document.getElementById("get_link");
    const call_btn = document.getElementById("call_btn");
    const get_name = document.getElementById("name");

    if (localStorage.getItem("name")) {
      document.getElementById("show_name").innerHTML =
        localStorage.getItem("name");

      get_name.value = localStorage.getItem("name");
    }

    get_name.addEventListener("keyup", function (e) {
      document.getElementById("show_name").innerHTML = get_name.value;
    });

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
      get_link.classList.add("loading");
      this.$axios.$post(process.env.SOCKET_URL_PORT + "/get_uuid").then((s) => {
        document.getElementById("input_link").value = s;
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
};
</script>