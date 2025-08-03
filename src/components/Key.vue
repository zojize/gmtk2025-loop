<script lang="ts" setup>
const { icon, disabled = false } = defineProps<{
  icon: string
  disabled?: boolean
}>()

const emit = defineEmits<{
  drag: [bounding: Omit<DOMRect, 'top' | 'right' | 'bottom' | 'left' | 'toJSON'>, icon: string]
  dragend: []
}>()

const dragging = ref(false)

const { x, y } = usePointer()
const dragEl = useTemplateRef('dragEl')
const bounding = useElementBounding(dragEl)

const offset = reactive({ x: 0, y: 0 })
function dragStart(event: PointerEvent) {
  if (disabled)
    return
  dragging.value = true
  offset.x = event.offsetX
  offset.y = event.offsetY
  const stop = useEventListener('pointermove', () => {
    emit('drag', {
      x: bounding.x.value,
      y: bounding.y.value,
      width: bounding.width.value,
      height: bounding.height.value,
    }, icon)
  })
  useEventListener('pointerup', () => {
    dragging.value = false
    stop()
    emit('dragend')
  }, { once: true })
}
</script>

<template>
  <div
    class="rounded-xl bg-gray-200 dark:bg-white/50 hover:bg-gray-300 dark:hover:bg-white/80"
    :class="dragging && 'opacity-20'"
    @pointerdown="dragStart"
  >
    <div class="text-gray-700 h-12 w-12 cursor-grab dark:text-white" :class="icon" />
  </div>
  <div
    v-if="dragging"
    ref="dragEl"
    class="rounded-xl bg-gray-200 fixed touch-none dark:bg-white/50 hover:bg-gray-300/50 dark:hover:bg-white/80"
    :style="{ left: `${x - offset.x}px`, top: `${y - offset.y}px` }"
  >
    <div class="text-gray-700 h-12 w-12 cursor-grab dark:text-white" :class="icon" />
  </div>
</template>
