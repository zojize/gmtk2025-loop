<script setup lang="ts">
import { decompressFromEncodedURIComponent as decode, compressToEncodedURIComponent as encode } from 'lz-string'

const { duration } = defineProps<{
  duration: number
}>()

const emit = defineEmits<{
  play: []
  pause: []
  restart: []
}>()

const paused = ref(true)
const progress = ref(0)
const accumulated = ref(0)
const sign = computed(() => Math.floor(accumulated.value / duration) % 2 === 0 ? 1 : -1)
let last = performance.now()
useRafFn(() => {
  if (!paused.value) {
    const now = performance.now()
    accumulated.value += now - (last ?? now)
    const offset = sign.value > 0 ? 0 : 100
    progress.value = sign.value * (accumulated.value % duration) / duration * 100 + offset
  }
  last = performance.now()
})

const timelineEl = useTemplateRef('timelineEl')
const timelineBounding = useElementBounding(timelineEl)
const preview = ref<{
  icon: string
  track: 0 | 1
  x: number
  width: number
  type: 'left' | 'up' | 'right'
} | undefined>()

type Rect = Omit<DOMRect, 'top' | 'right' | 'bottom' | 'left' | 'toJSON'>

const minWidth = ref(-1)
const searchParams = new URLSearchParams(window.location.search)
let itemsFromQuery: (NonNullable<typeof preview.value> & { active: boolean })[]
try {
  const timelineStr = searchParams.get('timeline')
  itemsFromQuery = timelineStr ? JSON.parse(decode(timelineStr)) : []
}
catch {
  itemsFromQuery = []
}
const timelineItems = ref<(NonNullable<typeof preview.value> & { active: boolean })[]>(itemsFromQuery ?? [])
watch(timelineItems, (items) => {
  const searchParams = new URLSearchParams(window.location.search)
  searchParams.set('timeline', encode(JSON.stringify(items)))
  window.history.replaceState({}, '', `?${searchParams.toString()}`)
}, { deep: true })
function onDrag(dragBounding: Rect, icon: string) {
  if (rectRectCollision(dragBounding, {
    x: timelineBounding.x.value,
    y: timelineBounding.y.value,
    width: timelineBounding.width.value,
    height: timelineBounding.height.value,
  })) {
    const width = (dragBounding.width / timelineBounding.width.value) * 100
    if (minWidth.value < 0)
      minWidth.value = width
    const newPreview = {
      icon: `#${icon.slice(2)}`,
      track: (dragBounding.y + dragBounding.height / 2) < (timelineBounding.y.value + timelineBounding.height.value / 2) ? 0 : 1,
      x: Math.min(100 - minWidth.value, Math.max(0, (dragBounding.x - timelineBounding.x.value) / timelineBounding.width.value * 100)),
      width,
      type: icon.slice(15) as 'left' | 'up' | 'right',
    } satisfies typeof preview.value

    if (!timelineItems.value.some((item) => {
      if (item.track === newPreview.track) {
        const itemEnd = item.x + item.width
        const dataEnd = newPreview.x + newPreview.width
        return (newPreview.x < itemEnd && dataEnd > item.x)
      }
      return false
    })) {
      preview.value = newPreview
    }
    else if (preview.value && Math.abs(newPreview.x - preview.value.x) > minWidth.value) {
      preview.value = undefined
    }
  }
  else {
    preview.value = undefined
  }
}

function onDragEnd() {
  if (!paused.value || accumulated.value > 0) {
    preview.value = undefined
    // TODO show a toast or something
    return
  }
  if (preview.value) {
    timelineItems.value.push({ ...preview.value, active: false })
  }
  preview.value = undefined
}

function rectRectCollision(rect1: Rect, rect2: Rect): boolean {
  return (
    rect1.x < rect2.x + rect2.width
    && rect1.x + rect1.width > rect2.x
    && rect1.y < rect2.y + rect2.height
    && rect1.y + rect1.height > rect2.y
  )
}

const timelineContext = {
  paused,
  sign,
  progress,
  timelineBounding,
  timelineItems,
  minWidth,
  accumulated,
}
export type TimelineContext = typeof timelineContext

const keys = computed(() => {
  const currentX = progress.value
  if (paused.value) {
    timelineItems.value.forEach((item) => {
      item.active = false
    })
    return { up: false, left: false, right: false }
  }
  return timelineItems.value.reduce((acc, item) => {
    if (item.x <= currentX && item.x + item.width >= currentX) {
      item.active = true
      let type = item.type
      if (sign.value < 0) {
        type = ({
          left: 'right',
          up: 'up',
          right: 'left',
        } as const)[type]
      }
      acc[type] = true
    }
    else {
      item.active = false
    }
    return acc
  }, { up: false, left: false, right: false })
})

function play() {
  paused.value = false
  emit('play')
}

function pause() {
  paused.value = true
  emit('pause')
}

function restart() {
  accumulated.value = 0
  progress.value = 0
  paused.value = true
  emit('restart')
}

function deleteItems() {
  if (accumulated.value > 0) {
    return
  }
  timelineItems.value = []
}

defineExpose({
  timelineContext,
  keys,
  play,
  pause,
  restart,
})
</script>

<template>
  <div class="mt-2 flex flex-row gap-2 justify-end">
    <button v-if="paused" class="icon-btn" title="Delete All Items" @click="deleteItems">
      <div class="i-carbon-trash-can" />
    </button>
    <button class="icon-btn" title="Play" :disabled="!paused" @click="play">
      <div class="i-carbon-play-filled-alt" />
    </button>
    <button class="icon-btn" title="Restart" @click="restart">
      <div class="i-carbon-restart" />
    </button>
  </div>
  <div class="mx-4 mt-2 p-2 border border-blue-400 bg-white max-h-48 dark:border-blue-600 dark:bg-gray-900">
    <svg ref="timelineEl" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 10" class="h-full w-full">
      <symbol id="carbon-arrow-left" viewBox="0 0 32 32"><!-- Icon from Carbon by IBM - undefined --><path fill="currentColor" d="m14 26l1.41-1.41L7.83 17H28v-2H7.83l7.58-7.59L14 6L4 16z" /></symbol>
      <symbol id="carbon-arrow-up" viewBox="0 0 32 32"><!-- Icon from Carbon by IBM - undefined --><path fill="currentColor" d="M16 4L6 14l1.41 1.41L15 7.83V28h2V7.83l7.59 7.58L26 14z" /></symbol>
      <symbol id="carbon-arrow-right" viewBox="0 0 32 32"><!-- Icon from Carbon by IBM - undefined --><path fill="currentColor" d="m18 6l-1.43 1.393L24.15 15H4v2h20.15l-7.58 7.573L18 26l10-10z" /></symbol>
      <line x1="-1000" y1="2.5" x2="1000" y2="2.5" stroke="currentColor" stroke-width="0.125" opacity="0.25" />
      <line x1="-1000" y1="7.5" x2="1000" y2="7.5" stroke="currentColor" stroke-width="0.125" opacity="0.25" />
      <template v-if="preview">
        <rect
          :x="preview.x"
          :y="preview.track === 0 ? 0 : 5"
          :width="preview.width"
          :height="5"
          class="fill-gray-200 dark:fill-gray-600"
          rx="1"
          ry="1"
        />
        <use :href="preview.icon" :x="preview.x" :y="preview.track === 0 ? 0 : 5" :width="preview.width" :height="5" />
      </template>
      <TimelineItem
        v-for="item in timelineItems"
        :key="`${item.track}-${item.x}-${item.width}`"
        :data="item"
        :context="timelineContext"
      />
      <line :x1="progress" y1="-100" :x2="progress" y2="100" stroke="currentColor" stroke-width="0.25" opacity="0.8" />
    </svg>
  </div>
  <div class="p-2 flex flex-row gap-2 justify-center">
    <Key icon="i-carbon-arrow-left" @drag="onDrag" @dragend="onDragEnd" />
    <Key icon="i-carbon-arrow-up" @drag="onDrag" @dragend="onDragEnd" />
    <Key icon="i-carbon-arrow-right" @drag="onDrag" @dragend="onDragEnd" />
  </div>
</template>
