<script lang="ts" setup>
const { copy } = useClipboard()

const helpDialog = useTemplateRef<HTMLDialogElement>('helpDialog')

function share() {
  copy(window.location.href)
}

function toggleHelp() {
  if (helpDialog.value?.matches(':popover-open')) {
    helpDialog.value.hidePopover()
  }
  else {
    helpDialog.value?.showPopover()
  }
}
</script>

<template>
  <!-- Help Modal -->
  <dialog
    ref="helpDialog"
    popover="auto"
    class="m-0 p-6 border-0 rounded-lg bg-white max-w-md shadow-xl left-1/2 top-1/2 dark:bg-gray-800 -translate-x-1/2 -translate-y-1/2"
  >
    <!-- Close button -->
    <button
      class="text-gray-500 right-4 top-4 absolute dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-200"
      @click="helpDialog?.hidePopover()"
    >
      <div class="i-carbon-close text-xl" />
    </button>

    <!-- Modal content -->
    <div class="pr-8">
      <h2 class="text-2xl text-gray-900 font-bold mb-4 dark:text-white">
        How to Play
      </h2>
      <div class="text-gray-700 space-y-3 dark:text-gray-300">
        Drag these keys
        <div class="my-2 flex gap-2">
          <Key icon="i-carbon-arrow-left" disabled />
          <Key icon="i-carbon-arrow-up" disabled />
          <Key icon="i-carbon-arrow-right" disabled />
        </div>
        from the bottom left corner onto the timeline to create actions.
        <br>
        Click the <span class="i-carbon-play-filled-alt icon-btn align-sub" /> button start the timeline loop. You can stretch and move the actions on the timeline to adjust their timing and order, and double-click an action to remove it.
      </div>
    </div>
  </dialog>

  <nav class="text-xl mr-2 mt-2 inline-flex gap-2 right-0 top-0 absolute z-10">
    <button class="icon-btn" title="Help" @click="toggleHelp">
      <div class="i-carbon-help" />
    </button>

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
  <main class="p-2 flex flex-col h-screen w-screen justify-center">
    <Game />
  </main>
</template>
