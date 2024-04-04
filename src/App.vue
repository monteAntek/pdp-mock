<template>
  <div class="pdp">
    <div class="main-wrapper">
      <div ref="productWrapper" class="product-wrapper">
        <div ref="gallery" class="gallery-wrapper">
          <div class="gallery">
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
        </div>
        <div class="bottomsheet-wrapper">
          <div
            ref="bottomsheet"
            :class="['bottomsheet', { 'bottomsheet--open': isBottomsheetOpen }]"
          >
            <button v-if="!isDesktop" type="button" @click="toggleBottomsheet">
              open / close
            </button>
            <h1>BOTTOMSHEET</h1>
          </div>
        </div>
      </div>
    </div>
    <div class="others-wrapper">
      <h1>OTHERS</h1>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch, toRefs } from "vue";
import { useMediaQuery, useEventListener, useScroll } from "@vueuse/core";

const isDesktop = useMediaQuery("(min-width: 1024px)");

const bottomsheet = ref<HTMLElement>();
const gallery = ref<HTMLElement>();
const isBottomsheetOpen = ref(false);
const bsTop = ref();
const lastImage = ref<Element>();
const lastImageBottom = ref();
const productWrapper = ref<HTMLElement>();

const { isScrolling, arrivedState, directions } = useScroll(productWrapper);
const { top: reachedTop } = toRefs(arrivedState);
const { top: isScrollingUp } = toRefs(directions);

function toggleBottomsheet() {
  isBottomsheetOpen.value = !isBottomsheetOpen.value;
  bottomsheet.value?.classList.toggle("bottomsheet--open");
}

watch(isBottomsheetOpen, () => {
  if (bottomsheet.value) {
    isBottomsheetOpen.value
      ? bottomsheet.value.classList.add("bottomsheet--open")
      : bottomsheet.value?.classList.remove("bottomsheet--open");
  }
});

onMounted(() => {
  useEventListener(gallery, "scroll", () => {
    lastImage.value = gallery.value?.querySelectorAll(
      ".gallery-image:last-child"
    )[0];

    lastImageBottom.value = lastImage.value?.getBoundingClientRect().bottom;

    bsTop.value = bottomsheet.value?.getBoundingClientRect().top;

    if (bsTop.value && lastImageBottom.value) {
      isBottomsheetOpen.value = lastImageBottom.value < bsTop.value;
    }
  });

  useEventListener(productWrapper, "scroll", () => {
    if (isScrolling.value && isScrollingUp.value && isBottomsheetOpen.value) {
      isBottomsheetOpen.value = reachedTop.value;
    }
  });
});
</script>

<style lang="scss" scoped>
.main-wrapper {
  min-height: 100vh;
  position: relative;

  @media only screen and (min-width: 1024px) {
    overflow: visible;
  }
}

.product-wrapper {
  display: flex;
  flex-direction: column;
  max-height: stretch;
  height: var(--app-height, 100vh);
  overflow: scroll;

  @media only screen and (min-width: 1024px) {
    flex-direction: row;
    position: relative;
    top: 0;
    max-height: none;
    height: auto;
    overflow: visible;
  }
}

.gallery-wrapper {
  min-height: 75vh;
  overscroll-behavior: none;
  overflow: scroll;
  flex-grow: 1;

  @media only screen and (min-width: 1024px) {
    min-height: 100vh;
    height: auto;
    width: 50%;
    overscroll-behavior: auto;
    overflow: visible;
  }
}

.gallery {
  background-color: blueviolet;
  height: fit-content;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
}

.gallery-image {
  width: 100%;
  height: 85vh;
  border: 1px solid red;
  display: flex;
  justify-content: center;
  align-items: center;

  @media only screen and (min-width: 1024px) {
    height: 100vh;
  }
}

.bottomsheet-wrapper {
  position: relative;
  background-color: chartreuse;
  @media only screen and (min-width: 1024px) {
    width: 50%;
    background-color: transparent;
  }
}

.bottomsheet {
  background-color: chartreuse;
  height: 790px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  padding: 50px;
  overflow: hidden;
  transition: all 0.3s;
  transform: translateY(0);

  @media only screen and (min-width: 1024px) {
    position: sticky;
    top: 0;
  }

  &--open {
    transform: translateY(-70%);
  }
}

.others-wrapper {
  background-color: chocolate;
  min-height: 500px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 50px;
}
</style>
