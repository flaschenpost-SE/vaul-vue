<script setup lang="ts">
import { computed, ref, watchEffect } from 'vue'
import { DialogContent } from 'reka-ui'
import { injectDrawerRootContext } from './context'
import { isVertical } from './helpers'
import { useScaleBackground } from './useScaleBackground'

const {
  open,
  isOpen,
  snapPointsOffset,
  hasSnapPoints,
  drawerRef,
  onPress,
  onDrag,
  onRelease,
  onCancel,
  isAllowedToDrag,
  modal,
  emitOpenChange,
  dismissible,
  keyboardIsOpen,
  closeDrawer,
  direction,
  handleOnly,
} = injectDrawerRootContext()

useScaleBackground()

const delayedSnapPoints = ref(false)

const snapPointHeight = computed(() => {
  if (snapPointsOffset.value && snapPointsOffset.value.length > 0)
    return `${snapPointsOffset.value[0]}px`

  return '0'
})

function handlePointerDownOutside(event: Event) {
  if (!modal.value || event.defaultPrevented) {
    event.preventDefault()
    return
  }
  if (keyboardIsOpen.value)
    keyboardIsOpen.value = false

  if (dismissible.value) {
    emitOpenChange(false)
  }
  else {
    event.preventDefault()
  }
}

function handlePointerDown(event: PointerEvent) {
  if (handleOnly.value)
    return

  onPress(event)
}

function handleOnDrag(event: PointerEvent) {
  if (handleOnly.value)
    return

  onDrag(event)
}

let touchStart: { x: number, y: number } | null = null

function handleTouchStart(event: TouchEvent) {
  const touch = event.touches[0]
  touchStart = touch ? { x: touch.clientX, y: touch.clientY } : null
}

// Once the drawer follows the finger, a native scroll of the same touch would
// cancel the drag (see onCancel). Only along the drag axis, so horizontal
// scrollers inside the drawer keep working.
function handleTouchMove(event: TouchEvent) {
  const touch = event.touches[0]
  if (!isAllowedToDrag.value || !event.cancelable || !touchStart || !touch)
    return

  const deltaX = Math.abs(touch.clientX - touchStart.x)
  const deltaY = Math.abs(touch.clientY - touchStart.y)
  if (isVertical(direction.value) ? deltaY > deltaX : deltaX > deltaY)
    event.preventDefault()
}

watchEffect (() => {
  if (hasSnapPoints.value) {
    window.requestAnimationFrame(() => {
      delayedSnapPoints.value = true
    })
  }
})
</script>

<template>
  <DialogContent
    ref="drawerRef"
    data-vaul-drawer=""
    :data-vaul-drawer-direction="direction"
    :data-vaul-delayed-snap-points="delayedSnapPoints ? 'true' : 'false'"
    :data-vaul-snap-points="isOpen && hasSnapPoints ? 'true' : 'false'"
    :style="{ '--snap-point-height': snapPointHeight }"
    @pointerdown="handlePointerDown"
    @pointermove="handleOnDrag"
    @pointerup="onRelease"
    @pointercancel="onCancel"
    @touchstart.passive="handleTouchStart"
    @touchmove="handleTouchMove"
    @pointer-down-outside="handlePointerDownOutside"
    @open-auto-focus.prevent
    @escape-key-down="(event) => {
      if (!dismissible)
        event.preventDefault()
    }"
  >
    <slot />
  </DialogContent>
</template>
