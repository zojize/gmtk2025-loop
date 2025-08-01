<script lang="ts" setup>
const { icon } = defineProps<{
  icon: string
}>()

const emit = defineEmits<{
  drag: [bounding: Omit<DOMRect, 'top' | 'right' | 'bottom' | 'left' | 'toJSON'>, icon: string]
  dragend: []
}>()

const dragging = ref(false)

const { x, y } = useMouse()
const dragEl = useTemplateRef('dragEl')
const bounding = useElementBounding(dragEl)

const offset = reactive({ x: 0, y: 0 })
function dragStart(event: MouseEvent) {
  dragging.value = true
  offset.x = event.offsetX
  offset.y = event.offsetY
  const stop = useEventListener('mousemove', () => {
    emit('drag', {
      x: bounding.x.value,
      y: bounding.y.value,
      width: bounding.width.value,
      height: bounding.height.value,
    }, icon)
  })
  useEventListener('mouseup', () => {
    dragging.value = false
    stop()
    emit('dragend')
  }, { once: true })
}
</script>

<template>
  <div
    class="rounded-xl dark:bg-white/50 dark:hover:bg-white/80"
    :class="dragging && 'opacity-20'"
    @mousedown="dragStart"
  >
    <div class="text-white h-12 w-12 cursor-grab" :class="icon" />
  </div>
  <div
    v-if="dragging"
    ref="dragEl"
    class="rounded-xl fixed touch-none dark:bg-white/50 dark:hover:bg-white/80"
    :style="{ left: `${x - offset.x}px`, top: `${y - offset.y}px` }"
  >
    <div class="text-white h-12 w-12 cursor-grab" :class="icon" />
  </div>
</template>
