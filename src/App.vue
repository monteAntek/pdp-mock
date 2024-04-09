<template>
  <div ref="pdp" :class="['pdp', { scroll: isBottomsheetOpen && !isDesktop }]">
    <div ref="productWrapper" class="product-wrapper">
      <div
        ref="gallery"
        :class="['gallery', { hidden: isBottomsheetOpen && !isDesktop }]"
      >
        <div class="gallery-image">
          <p>IMAGE 1</p>
        </div>
        <div class="gallery-image">
          <p>IMAGE 2</p>
        </div>
        <div class="gallery-image">
          <p>IMAGE 3</p>
        </div>
        <div class="gallery-image">
          <p>IMAGE 4</p>
        </div>
        <div class="gallery-image">
          <p>IMAGE 5</p>
        </div>
        <div class="gallery-image">
          <p>IMAGE 6</p>
        </div>
      </div>
      <div
        ref="bottomsheet"
        :class="[
          'bottomsheet',
          { 'bottomsheet--open': isBottomsheetOpen && !isDesktop },
        ]"
      >
        <button
          v-if="!isDesktop"
          class="drag-handle-container"
          @click.self="toggleBottomsheet"
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
import { useMediaQuery, useScroll, useSwipe } from "@vueuse/core";

const isDesktop = useMediaQuery("(min-width: 1024px)");

const pdp = ref<HTMLElement>();
const bottomsheet = ref<HTMLElement>();
const gallery = ref<HTMLElement>();
const isBottomsheetOpen = ref(false);
const productWrapper = ref<HTMLElement>();

const { y: galleryScroll, arrivedState: galleryArrivedState } =
  useScroll(gallery);
const { bottom: hasGalleryReachedBottom } = toRefs(galleryArrivedState);

const { y: pdpScroll, arrivedState: pdpArrivedState } = useScroll(pdp);
const { top: hasPdpReachedTop } = toRefs(pdpArrivedState);

const {
  isSwiping: isBottomsheetSwiping,
  direction: bottomsheetSwipeDirection,
} = useSwipe(bottomsheet);

watch(isBottomsheetSwiping, () => {
  if (!isBottomsheetOpen.value && bottomsheetSwipeDirection.value === "up") {
    toggleBottomsheet();
  }

  if (isBottomsheetOpen.value && bottomsheetSwipeDirection.value === "down") {
    toggleBottomsheet();
  }
});

watch(hasGalleryReachedBottom, () => {
  if (hasGalleryReachedBottom.value && !isBottomsheetOpen.value) {
    toggleBottomsheet();
  }
});

watch(hasPdpReachedTop, () => {
  if (hasPdpReachedTop.value && isBottomsheetOpen.value) {
    toggleBottomsheet();
  }
});

function toggleBottomsheet() {
  if (!isDesktop.value) {
    isBottomsheetOpen.value = !isBottomsheetOpen.value;
    bottomsheet.value?.classList.toggle("bottomsheet--open");
    pdpScroll.value += 2;
    galleryScroll.value -= 2;
  }
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
  transition: all 0.3s;
  transform: translateY(0);

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
</style>
