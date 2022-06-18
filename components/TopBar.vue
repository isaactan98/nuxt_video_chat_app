<template>
  <div class="navbar bg-base-100">
    <div class="flex-1">
      <nuxt-link to="/" class="btn btn-ghost normal-case text-xl"
        >Nuxt Video Chat App</nuxt-link
      >
    </div>
    <div class="flex-none dropdown dropdown-end">
      <button class="btn btn-square btn-ghost" tabindex="0">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          class="inline-block w-5 h-5 stroke-current"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M5 12h.01M12 12h.01M19 12h.01M6 12a1 1 0 11-2 0 1 1 0 012 0zm7 0a1 1 0 11-2 0 1 1 0 012 0zm7 0a1 1 0 11-2 0 1 1 0 012 0z"
          ></path>
        </svg>
      </button>
      <ul
        tabindex="0"
        class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52"
      >
        <li><a>Item 1</a></li>
        <li><a>Item 2</a></li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {};
</script>

<style>
</style>