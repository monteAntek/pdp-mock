<template>
  <div ref="pdp" :class="['pdp', { scroll: isBottomsheetOpen && !isDesktop }]">
    <div class="product-wrapper">
      <div
        ref="gallery"
        :class="['gallery', { hidden: isBottomsheetOpen && !isDesktop }]"
      >
        <div class="gallery-image" v-for="image in images" :key="image.id">
          <p>{{ image.text }}</p>
        </div>
      </div>
      <div
        ref="bottomsheet"
        :class="[
          'bottomsheet',
          {
            'bottomsheet--open': !isDesktop && isBottomsheetOpen,
            dragging: isDragged,
          },
        ]"
      >
        <button
          ref="dragHandle"
          v-if="!isDesktop"
          class="drag-handle-container"
          @click.prevent="toggleBottomsheet"
          @mousedown="onDragStart"
          @mousemove.passive="onDragMove"
          @mouseup="onDragEnd"
          @touchstart="onDragStart"
          @touchmove="onDragMove"
          @touchend="onDragEnd"
        >
          <div class="drag-handle" />
        </button>
        <h1>BOTTOMSHEET</h1>
      </div>
    </div>
    <div class="others-wrapper">
      <h1>OTHERS</h1>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, toRefs, watch } from "vue";
import { useMediaQuery, useScroll } from "@vueuse/core";

const isDesktop = useMediaQuery("(min-width: 1024px)");

const pdp = ref<HTMLElement>();
const bottomsheet = ref<HTMLElement>();
const gallery = ref<HTMLElement>();
const dragHandle = ref<HTMLElement>();

const isClicked = ref(false);
const isDragged = ref(false);
const tapPosition = ref(0);
const isBottomsheetOpen = ref(false);
const dragDistance = ref(0);

const images = [
  { id: 1, text: "IMAGE 1" },
  { id: 2, text: "IMAGE 2" },
  { id: 3, text: "IMAGE 3" },
  { id: 4, text: "IMAGE 4" },
  { id: 5, text: "IMAGE 5" },
  { id: 6, text: "IMAGE 6" },
];

const { y: galleryScroll, arrivedState: galleryArrivedState } =
  useScroll(gallery);
const { bottom: hasGalleryReachedBottom } = toRefs(galleryArrivedState);

const { y: pdpScroll, arrivedState: pdpArrivedState } = useScroll(pdp);
const { top: hasPdpReachedTop } = toRefs(pdpArrivedState);

watch(hasGalleryReachedBottom, () => {
  if (
    !isDragged.value &&
    !isBottomsheetOpen.value &&
    hasGalleryReachedBottom.value
  ) {
    toggleBottomsheet();
  }
});

watch(hasPdpReachedTop, () => {
  if (!isDragged.value && isBottomsheetOpen.value && hasPdpReachedTop.value) {
    toggleBottomsheet();
  }
});

function toggleBottomsheet() {
  if (!isDesktop.value && !isDragged.value) {
    isBottomsheetOpen.value = !isBottomsheetOpen.value;
    bottomsheet.value?.classList.toggle("bottomsheet--open");

    pdpScroll.value = Math.max(0, pdpScroll.value + 1);
    galleryScroll.value = Math.max(0, galleryScroll.value - 1);
  }
}

function isHandledByTouch() {
  return "ontouchstart" in document.documentElement;
}

function checkIfIsDragging(
  initialPosition: number,
  newPosition: number
): boolean {
  const distance = Math.abs(newPosition - initialPosition);

  return distance >= 10;
}

function getDragPositionY(event: MouseEvent | TouchEvent): { y: number } {
  let y = 0;

  if (isHandledByTouch()) {
    y = (event as TouchEvent)?.changedTouches?.[0]?.clientY;
  } else {
    y = (event as MouseEvent)?.clientY;
  }

  return { y };
}

function onDragStart(event: MouseEvent | TouchEvent) {
  if (isHandledByTouch()) {
    isClicked.value = true;
    isDragged.value = false;

    tapPosition.value = getDragPositionY(event).y;
    console.log("tap position on drag start", tapPosition.value);
  } else {
    const mouseButton = (event as MouseEvent).button;

    if (mouseButton === 0) {
      isClicked.value = true;
      isDragged.value = false;
    }
  }
}

function onDragMove(event: MouseEvent | TouchEvent) {
  if (isClicked.value) {
    const newPosition = getDragPositionY(event).y;

    console.log("tap position on drag move", tapPosition.value);
    console.log("new position on drag move", newPosition);

    if (checkIfIsDragging(tapPosition.value, newPosition)) {
      isDragged.value = true;
      const realDragDistance = tapPosition.value - newPosition;
      dragDistance.value = Math.abs(tapPosition.value - newPosition);
      console.log("drag distance", dragDistance.value);
      console.log("real drag distance", realDragDistance);

      const translateValue = dragDistance.value * -1;
      console.log("translate value", translateValue);
      document.body.style.setProperty(
        "--handleDragDistanceY",
        `${translateValue}px`
      );

      if (
        bottomsheet.value &&
        dragDistance.value >
          bottomsheet.value?.getBoundingClientRect().height * 0.25 &&
        realDragDistance > 0
      ) {
        isBottomsheetOpen.value = true;
      } else if (
        bottomsheet.value &&
        dragDistance.value >
          bottomsheet.value?.getBoundingClientRect().height * 0.25 &&
        realDragDistance < 0
      ) {
        isBottomsheetOpen.value = false;
      }
    }
  }

  if (!isDragged.value) {
    return;
  }
}

function onDragEnd() {
  console.log("tap position on drag end", tapPosition.value);

  isClicked.value = false;
  isDragged.value = false;
  tapPosition.value = 0;
}
</script>

<style lang="scss" scoped>
.pdp {
  max-height: 100vh;
  max-height: 100dvh;
  overflow-y: hidden;

  @media only screen and (min-width: 1024px) {
    max-height: none;
    min-height: 100vh;
    height: auto;
    overflow: visible;
  }
}

.product-wrapper {
  max-height: 100vh;
  overflow-y: hidden;

  @media only screen and (min-width: 1024px) {
    display: flex;
    overflow: visible;
    max-height: none;
    height: 100%;
    width: 100%;
  }
}

.gallery {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  max-height: 80vh;
  overflow: scroll;

  @media only screen and (min-width: 1024px) {
    max-height: none;
    width: 50%;
    overflow: visible;
  }
}

.gallery-image {
  background-color: blueviolet;
  width: 100%;
  min-height: 100vh;
  border: 1px solid red;
  display: flex;
  justify-content: center;
  align-items: center;
}

.bottomsheet {
  background-color: chartreuse;
  height: 790px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  overflow: hidden;
  transform: translateY(0);
  transition: all 0.3s;

  @media only screen and (min-width: 1024px) {
    width: 50%;
    position: sticky;
    top: 0;
  }

  &--open {
    transform: translateY(-70%);
  }
}

.drag-handle-container {
  all: unset;
  width: 100%;
  padding-block: 14px;
}

.drag-handle {
  margin-left: auto;
  margin-right: auto;
  height: 2px;
  width: 50px;
  border-radius: 0.125rem;
  background-color: rgb(0 0 0);
}

.others-wrapper {
  background-color: chocolate;
  min-height: 500px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 50px;
}

.scroll {
  overflow-y: scroll;
}

.hidden {
  overflow-y: hidden;
}

.dragging {
  transform: translateY(var(--handleDragDistanceY));
}
</style>
