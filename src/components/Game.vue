<script lang="ts" setup>
import type Level from './Level.vue'
import type Timeline from './Timeline.vue'

const timeline = useTemplateRef<InstanceType<typeof Timeline>>('timeline')

const currentLevel = ref(1)
const collectedGoals = ref(0)
const levelsData = [
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
      { x: -6, y: 1, width: 1, height: 1 },
      { x: 6, y: 1, width: 1, height: 1 },
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
] satisfies Omit<InstanceType<(typeof Level)>['$props'], 'timeline'>[]

const levels = useTemplateRef('levels')

whenever(() => collectedGoals.value === currentLevel.value, () => {
  currentLevel.value++
  collectedGoals.value = 0
  timeline.value!.restart()
  timeline.value!.pause()
  levels.value!.forEach((level) => {
    level!.restart()
  })
  if (currentLevel.value > levelsData.length) {
    currentLevel.value = 1
  }
})
</script>

<template>
  <div class="flex flex-row flex-wrap justify-center relative children:flex-basis-[33%]">
    <Level
      v-for="({ characterStartPos, goalPos, obstacles }, key) in levelsData.slice(0, currentLevel)"
      :key
      ref="levels"
      :character-start-pos
      :goal-pos
      :obstacles
      :timeline
      @goal="collectedGoals++"
    />
  </div>
  <Timeline ref="timeline" :duration="2000" />
</template>
