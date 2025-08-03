<script lang="ts" setup>
import type Level from './Level.vue'
import type Timeline from './Timeline.vue'
import confetti from 'canvas-confetti'

const timeline = useTemplateRef<InstanceType<typeof Timeline>>('timeline')

const searchParams = new URLSearchParams(window.location.search)
const currentLevel = ref(searchParams.get('level') ? +searchParams.get('level')! : 1)
const collectedGoals = ref(0)
const gameCompleted = ref(false)
const gameStarted = ref(false)
const spikeDirections = {
  up: 'rotate(0deg)',
  right: 'rotate(90deg)',
  down: 'rotate(180deg)',
  left: 'rotate(270deg)',
}
const levelsData: Omit<InstanceType<(typeof Level)>['$props'], 'timeline'>[] = [
  {
    characterStartPos: { x: -8, y: 1.25 },
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 1.25 },
    obstacles: [
      { x: 0, y: 1, width: 4, height: 1 },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 1.25 },
    obstacles: [
      { x: -5, y: 1, width: 1, height: 1 },
      { x: 5, y: 1, width: 1, height: 1 },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 7.25 },
    obstacles: [
      { x: -8, y: 5, width: 4, height: 1 },
    ],
    goalPos: { x: -8, y: 1.25 },
  },
  {
    characterStartPos: { x: 8, y: 17.25 },
    obstacles: [
      { x: 7, y: 15, width: 3, height: 1 },
      { x: -4, y: 2, width: 1, height: 3 },
      { x: -2, y: 4, width: 3, height: 1 },
      { x: 0, y: 6, width: 1, height: 3 },
      { x: 2, y: 8, width: 3, height: 1 },
    ],
    spikes: [
      { x: 0, y: 1, rotation: spikeDirections.up },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 12 },
    obstacles: [
      { x: -8, y: 8, width: 2, height: 2 },
      { x: -1, y: 2, width: 3, height: 2 },
      { x: 8, y: 8, width: 2, height: 1 },
    ],
    spikes: [
      { x: -8, y: 1, rotation: spikeDirections.up },
    ],
    goalPos: { x: 8, y: 10 },
  },
  {
    characterStartPos: { x: -8, y: 12.25 },
    obstacles: [
      { x: -6, y: 2, width: 1, height: 3 },
      { x: -8, y: 4, width: 3, height: 1 },
      { x: -8, y: 10, width: 14, height: 1 },
      { x: 0, y: 7, width: 1, height: 3 },
      { x: 6, y: 2, width: 1, height: 3 },
      { x: 8, y: 4, width: 3, height: 1 },
    ],
    goalPos: { x: -8, y: 6.25 },
  },
  {
    characterStartPos: { x: 0, y: 1.25 },
    obstacles: [
      { x: 1, y: 4, width: 3, height: 1 },
      { x: -1, y: 10, width: 3, height: 1 },
    ],
    goalPos: { x: -1, y: 14 },
  },
  {
    characterStartPos: { x: 9, y: 1.25 },
    obstacles: [
      { x: -4, y: 3, width: 2, height: 3 },
      { x: -8, y: 9, width: 2, height: 3 },
      { x: 4, y: 3, width: 1, height: 5 },
      { x: 6, y: 4, width: 2, height: 1 },
      { x: 4, y: 16, width: 1, height: 5 },
    ],
    goalPos: { x: -8, y: 14 },
  },
]

const levels = useTemplateRef('levels')

function onGoal() {
  collectedGoals.value++
}

whenever(() => collectedGoals.value === 9, () => {
  gameCompleted.value = true

  // Fire confetti celebration!
  const duration = 3 * 1000
  const animationEnd = Date.now() + duration
  const defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 0 }

  function randomInRange(min: number, max: number) {
    return Math.random() * (max - min) + min
  }

  const interval: NodeJS.Timeout = setInterval(() => {
    const timeLeft = animationEnd - Date.now()

    if (timeLeft <= 0) {
      return clearInterval(interval)
    }

    const particleCount = 50 * (timeLeft / duration)

    // Fire from left side
    confetti({
      ...defaults,
      particleCount,
      origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 },
    })

    // Fire from right side
    confetti({
      ...defaults,
      particleCount,
      origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 },
    })
  }, 250)
})

whenever(() => collectedGoals.value === currentLevel.value && collectedGoals.value < 9, () => {
  currentLevel.value++
  if (currentLevel.value > levelsData.length) {
    currentLevel.value = 1
  }
  resetLevels()
})

function resetLevels(restartTimeline: boolean = true) {
  collectedGoals.value = 0
  if (restartTimeline) {
    timeline.value!.restart()
  }
  levels.value!.forEach((level) => {
    level!.restart()
  })
}

function restartGame() {
  gameCompleted.value = false
  gameStarted.value = false
  currentLevel.value = 1
  resetLevels()
}

function startGame() {
  gameStarted.value = true
}

async function copySolution() {
  try {
    await navigator.clipboard.writeText(window.location.href)
    // You could add a toast notification here if needed
  }
  catch (err) {
    console.error('Failed to copy solution URL:', err)
  }
}

const levelsClass = [
  '',
  '',
  'grid grid-cols-2 grid-rows-1',
  'grid grid-cols-3 grid-rows-1',
  'grid grid-cols-2 grid-rows-2',
  // fixme: unocss should be able to handle nth-[-n+3]:children:(self-end col-span-2)
  'grid grid-cols-6 grid-rows-2 children:col-span-2 nth-1:children:self-end nth-2:children:self-end nth-3:children:self-end nth-4:children:(self-start col-[2/span_2]) nth-5:children:self-start',
  'grid grid-cols-3 grid-rows-2 nth-1:children:self-end nth-2:children:self-end nth-3:children:self-end nth-4:children:self-start nth-5:children:self-start nth-6:children:self-start',
  'grid grid-cols-3 grid-rows-3 nth-7:children:col-2',
  'grid grid-cols-6 grid-rows-3 children:col-span-2 !nth-7:children:col-[2/span_2]',
  'grid grid-cols-3 grid-rows-3',
]
</script>

<template>
  <!-- Start Screen -->
  <div
    v-if="!gameStarted"
    class="text-white flex flex-col items-center inset-0 justify-center fixed from-blue-400 to-pink-500 via-purple-500 bg-linear-to-br dark:from-indigo-900 dark:to-pink-900 dark:via-purple-900"
  >
    <div class="text-center animate-fade-in">
      <!-- Game Logo/Title -->
      <div class="mb-2">
        <h1 class="text-8xl text-transparent font-black mb-4 from-yellow-300 to-orange-400 bg-linear-to-r bg-clip-text drop-shadow-lg dark:from-cyan-300 dark:to-blue-400">
          A GAME THAT LOOPS
        </h1>
        <p class="text-xl text-white/90 mx-auto max-w-md dark:text-cyan-100/90">
          GMTK 2025 Game Jam entry
        </p>
        <p class="text-xl text-white/1 mx-auto max-w-md dark:text-cyan-100/1">
          What happens when webdev makes a game
        </p>
      </div>

      <!-- Menu Buttons -->
      <button
        class="text-3xl text-white font-bold px-12 py-6 rounded-full cursor-pointer transition-all duration-300 from-green-500 to-emerald-600 bg-linear-to-r hover:shadow-2xl hover:shadow-green-500/50 dark:from-emerald-400 dark:to-green-500 hover:-translate-y-2"
        @click="startGame"
      >
        PLAY
      </button>

      <!-- Decorative game elements -->
      <svg class="h-16 w-16 left-8 top-8 absolute animate-pulse" viewBox="0 0 100 100">
        <!-- Red spike triangle -->
        <polygon points="20,80 80,80 50,20" class="fill-red-500 dark:fill-red-400" />
      </svg>

      <svg class="h-12 w-12 right-12 top-16 absolute" viewBox="0 0 100 100">
        <!-- Green goal circle -->
        <circle cx="50" cy="50" r="35" class="fill-green-500 dark:fill-green-400" />
      </svg>

      <svg class="h-20 w-20 bottom-12 left-8 absolute animate-bounce" style="animation-delay: 1s;" viewBox="0 0 100 100">
        <!-- Blue player capsule -->
        <rect x="30" y="20" width="40" height="60" rx="20" ry="20" class="fill-blue-400 dark:fill-blue-300" />
      </svg>

      <svg class="h-14 w-14 bottom-12 right-8 absolute" style="animation-delay: 2s;" viewBox="0 0 100 100">
        <!-- Gray wall rectangle -->
        <rect x="20" y="20" width="60" height="60" class="fill-gray-600 dark:fill-gray-400" />
      </svg>
    </div>
  </div>

  <!-- Victory Screen -->
  <div
    v-else-if="gameCompleted"
    class="text-gray-900 flex flex-col items-center inset-0 justify-center fixed from-yellow-300 to-pink-400 via-orange-300 bg-linear-to-br dark:text-white dark:from-purple-900 dark:to-blue-900 dark:via-indigo-900"
  >
    <div class="text-center animate-fade-in">
      <div class="text-8xl mb-8 select-none animate-bounce">
        🎉
      </div>
      <h1 class="text-6xl text-transparent font-bold mb-4 from-orange-600 to-pink-600 bg-linear-to-r bg-clip-text dark:from-cyan-300 dark:to-purple-300">
        CONGRATULATIONS!
      </h1>
      <h2 class="text-3xl text-gray-800 mb-8 dark:text-cyan-100">
        You completed all 9 levels!
      </h2>
      <div class="flex flex-col gap-4">
        <button
          class="text-2xl text-white font-bold px-8 py-4 rounded-full cursor-pointer transition-all from-blue-500 to-purple-600 bg-linear-to-r hover:shadow-2xl dark:from-cyan-500 dark:to-purple-600 dark:hover:shadow-purple-500/25"
          @click="restartGame"
        >
          Play Again
        </button>
        <button
          class="text-lg text-gray-800 font-semibold px-6 py-3 rounded-full bg-white/95 cursor-pointer transition-all dark:text-cyan-100 dark:border dark:border-cyan-400/50 dark:bg-white/10 hover:bg-white hover:shadow-lg dark:hover:border-cyan-400/70 dark:hover:bg-white/20"
          @click="copySolution"
        >
          📋 Copy Solution URL
        </button>
      </div>
    </div>
  </div>

  <!-- Game Screen -->
  <div v-else-if="gameStarted && !gameCompleted" class="mx-auto h-[90cqmin] aspect-ratio-square items-center justify-center children:h-fit" :class="levelsClass[currentLevel]">
    <Level
      v-for="({ characterStartPos, goalPos, obstacles, spikes }, key) in levelsData.slice(0, currentLevel)"
      :key
      ref="levels"
      :character-start-pos
      :goal-pos
      :obstacles
      :spikes
      :timeline
      @goal="onGoal"
      @spike="resetLevels"
    />
  </div>
  <Timeline v-if="gameStarted && !gameCompleted" ref="timeline" :duration="2000" @restart="resetLevels(false)" />
</template>
