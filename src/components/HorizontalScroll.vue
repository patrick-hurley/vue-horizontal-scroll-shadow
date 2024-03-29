<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
const horizontalScrollWrapper = ref<HTMLInputElement | null>(null)
const horizontalScrollContent = ref<HTMLInputElement | null>(null)
const showBoxShadowLeft = ref(false)
const showBoxShadowRight = ref(false)

/*
Apply a box shadow to the left of right of the container depending
on whether there is a scrollbar visible, and based on its positioning
*/
function updateBoxShadow() {
  if (horizontalScrollWrapper.value) {
    const scroll_pos = horizontalScrollWrapper.value.scrollLeft
    const horizontalScrollWidth =
      horizontalScrollWrapper.value.scrollWidth - horizontalScrollWrapper.value.clientWidth
    if (scroll_pos === 0) {
      showBoxShadowLeft.value = false
    }
    if (scroll_pos > 10) {
      showBoxShadowLeft.value = true
    }
    if (scroll_pos < horizontalScrollWidth) {
      showBoxShadowRight.value = true
    }
    if (scroll_pos > horizontalScrollWidth - 10) {
      showBoxShadowRight.value = false
    }
  }
}

onMounted(() => {
  if (horizontalScrollWrapper.value && horizontalScrollContent.value) {
    // On table scroll
    horizontalScrollWrapper.value.addEventListener('scroll', updateBoxShadow)
    // On page resize
    window.addEventListener('resize', updateBoxShadow)
    // On page load and content change
    const resizeObserver = new ResizeObserver(updateBoxShadow)
    resizeObserver.observe(horizontalScrollContent.value)
  }
})

onUnmounted(() => {
  window.removeEventListener('resize', updateBoxShadow)
})
</script>

<template>
  <div class="horizontal-scroll">
    <div
      ref="horizontalScrollWrapper"
      class="horizontal-scroll--wrapper"
      :class="{ 'box-shadow-left': showBoxShadowLeft, 'box-shadow-right': showBoxShadowRight }"
    >
      <div ref="horizontalScrollContent" class="horizontal-scroll--content">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<style>
.horizontal-scroll {
  position: relative;
  overflow-x: hidden;
}
.horizontal-scroll--wrapper {
  display: flex;
  overflow-x: auto;
}
.horizontal-scroll--wrapper.box-shadow-right:after,
.horizontal-scroll--wrapper.box-shadow-left:before {
  content: '';
  /* deduct scroll bar height */
  height: calc(100% - 1em);
  width: 10px;
  top: 0;
  position: absolute;
  z-index: 1;
}
.horizontal-scroll--wrapper.box-shadow-right:after {
  right: 0;
  background: linear-gradient(to right, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.3));
}
.horizontal-scroll--wrapper.box-shadow-left:before {
  left: 0;
  background: linear-gradient(to right, rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0));
}
.horizontal-scroll--content {
  flex-grow: 1;
}
</style>
