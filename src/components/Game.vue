<script lang="ts" setup>
import type Level from './Level.vue'
import type Timeline from './Timeline.vue'

const timeline = useTemplateRef<InstanceType<typeof Timeline>>('timeline')

const currentLevel = ref(1)
const collectedGoals = ref(0)
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
  {
    characterStartPos: { x: -8, y: 1.25 },
    spikes: [
      { x: -5, y: 1, rotation: spikeDirections.up },
      { x: 5, y: 1, rotation: spikeDirections.up },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 1.25 },
    spikes: [
      { x: -5, y: 1, rotation: spikeDirections.up },
      { x: 5, y: 1, rotation: spikeDirections.up },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 1.25 },
    spikes: [
      { x: -5, y: 1, rotation: spikeDirections.up },
      { x: 5, y: 1, rotation: spikeDirections.up },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 1.25 },
    spikes: [
      { x: -5, y: 1, rotation: spikeDirections.up },
      { x: 5, y: 1, rotation: spikeDirections.up },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
  {
    characterStartPos: { x: -8, y: 1.25 },
    spikes: [
      { x: -5, y: 1, rotation: spikeDirections.up },
      { x: 5, y: 1, rotation: spikeDirections.up },
    ],
    goalPos: { x: 8, y: 1.25 },
  },
]

const levels = useTemplateRef('levels')

function onGoal() {
  collectedGoals.value++
}

whenever(() => collectedGoals.value === currentLevel.value, () => {
  currentLevel.value++
  if (currentLevel.value > levelsData.length) {
    currentLevel.value = 1
  }
  resetLevels()
})

function resetLevels() {
  collectedGoals.value = 0
  timeline.value!.restart()
  timeline.value!.pause()
  levels.value!.forEach((level) => {
    level!.restart()
  })
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
  <div class="mx-auto h-[100cqmin] aspect-ratio-square items-center justify-center children:h-fit" :class="levelsClass[currentLevel]">
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
  <Timeline ref="timeline" :duration="2000" />
</template>
