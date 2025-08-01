<template>
  <main class="p-2 flex flex-col h-screen w-screen">
    <nav class="text-xl mr-2 mt-2 inline-flex gap-2 right-0 absolute">
      <span class="dark:hidden">I only made dark mode ui for now... please switch if you are in light mode </span>
      <button class="icon-btn" @click="toggleDark()">
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
    <Game />
  </main>
</template>
