<script lang="ts" setup>
import type { TimelineContext } from './Timeline.vue'

const props = defineProps<{
  data: TimelineContext['timelineItems']['value'][number]
  context: TimelineContext
}>()
const { data, context } = props

function onMouseDownLeft() {
  const initialX = data.x
  const initialWidth = data.width
  const minX = context.timelineItems.value
    .reduce((acc, item) => {
      if (item.track === data.track) {
        const right = item.x + item.width
        if (right < data.x && right > acc) {
          return right
        }
      }
      return acc
    }, 0)
  const stop = useEventListener('mousemove', (e: MouseEvent) => {
    const timelineBounding = context.timelineBounding
    const xBefore = data.x
    data.x = Math.max(minX, Math.min(100, (e.clientX - timelineBounding.x.value) / timelineBounding.width.value * 100))
    data.width = initialX - data.x + initialWidth
    if (data.width < context.minWidth.value) {
      data.width = context.minWidth.value
      data.x = xBefore
    }
  })
  useEventListener('mouseup', () => {
    stop()
  }, { once: true })
}

function onMouseDownRight() {
  const maxWidth = context.timelineItems.value
    .reduce((acc, item) => {
      if (item.track === data.track) {
        const left = item.x
        if (left > data.x && left < acc) {
          return left
        }
      }
      return acc
    }, 100) - data.x
  const stop = useEventListener('mousemove', (e: MouseEvent) => {
    const timelineBounding = context.timelineBounding
    data.width = Math.max(context.minWidth.value, Math.min(maxWidth, (e.clientX - timelineBounding.x.value) / timelineBounding.width.value * 100 - data.x))
  })
  useEventListener('mouseup', () => {
    stop()
  }, { once: true })
}

function onGrab(event: MouseEvent) {
  const timelineBounding = context.timelineBounding
  const offsetX = (event.clientX - (timelineBounding.x.value + (data.x / 100) * timelineBounding.width.value)) / timelineBounding.width.value * 100
  const initialWidth = data.width
  const hasOverlap = () => {
    return context.timelineItems.value.some((item) => {
      if (item === data)
        return false
      if (item.track === data.track) {
        const itemEnd = item.x + item.width
        const dataEnd = data.x + data.width
        return (data.x < itemEnd && dataEnd > item.x)
      }
      return false
    })
  }
  const stop = useEventListener('mousemove', (e: MouseEvent) => {
    const xBefore = data.x
    const trackBefore = data.track
    data.x = Math.max(0, Math.min(100 - initialWidth, (e.clientX - timelineBounding.x.value) / timelineBounding.width.value * 100 - offsetX))
    data.track = (e.clientY - timelineBounding.y.value) < (timelineBounding.height.value / 2) ? 0 : 1
    if (hasOverlap()) {
      data.x = xBefore
      data.track = trackBefore
    }
  })
  useEventListener('mouseup', () => {
    stop()
  }, { once: true })
}

function removeSelf() {
  const index = context.timelineItems.value.indexOf(data)
  if (index !== -1) {
    context.timelineItems.value.splice(index, 1)
  }
}
</script>

<template>
  <rect
    :x="data.x"
    :y="data.track === 0 ? 0 : 5 + 0.05"
    :width="data.width"
    :height="4.9"
    class="cursor-grab"
    :class="{
      'opacity-100': data.active,
      'opacity-50': !data.active,
    }"
    stroke="white"
    stroke-width="0.1"
    fill="gray"
    opacity="0.5"
    rx="1"
    ry="1"
    @mousedown="onGrab"
    @dblclick="removeSelf"
  />
  <use
    :href="data.icon"
    class="cursor-grab transition-transform transition-ease-in-out"
    :x="data.x"
    :y="data.track === 0 ? 0 : 5"
    :width="data.width"
    :height="5"
    :transform-origin="`${data.x + data.width / 2} ${data.track === 0 ? 2.5 : 7.5}`"
    :style="{ transform: context.sign.value > 0 ? 'scaleX(1)' : 'scaleX(-1)' }"
    @mousedown="onGrab"
    @dblclick="removeSelf"
  />
  <rect
    :x="data.x - 0.5"
    :y="data.track === 0 ? 0 : 5"
    width="1"
    :height="5"
    class="cursor-w-resize"
    fill="transparent"
    @mousedown="onMouseDownLeft"
  />
  <rect
    :x="data.x + data.width - 0.5"
    :y="data.track === 0 ? 0 : 5"
    width="1"
    :height="5"
    class="cursor-e-resize"
    fill="transparent"
    @mousedown="onMouseDownRight"
  />
</template>
