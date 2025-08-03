<script lang="ts" setup>
const { copy } = useClipboard()

function share() {
  copy(window.location.href)
}
</script>

<template>
  <nav class="text-xl mr-2 mt-2 inline-flex gap-2 right-0 absolute z-10">
    <button class="icon-btn" title="Share" @click="share">
      <div class="i-carbon-share" />
    </button>

    <button class="icon-btn" title="Toggle Dark Mode" @click="toggleDark()">
      <div class="i-carbon-sun dark:i-carbon-moon" />
    </button>

    <a
      class="i-carbon-logo-github icon-btn"
      rel="noreferrer"
      href="https://github.com/zojize/gmtk2025-loop"
      target="_blank"
      title="GitHub"
    />
  </nav>
  <main class="p-2 flex flex-col h-screen w-screen">
    <Game />
  </main>
</template>
