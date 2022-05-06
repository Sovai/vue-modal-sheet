<template>
  <button @click="selectedProp === spring">Spring</button>
  <button @click="selectedProp = keyFrame">Keyframe</button>
  <div class="sheet-wrapper" :data-open="state === 'open' ? 1 : 0">
    <div class="sheet-overlay-bg" @click="setClose()"></div>
    <div class="modal-sheet" ref="sheetRef">
      <div class="bar"></div>
      <div class="card">
        <slot name="header">
          <div class="flex justify-space-between align-center st-sticky-header">
            <div>
              <div v-if="title" class="sheet-title">{{ title }}</div>
              <div v-if="subtitle" class="sheet-subtitle st-margin-16-top">
                {{ subtitle }}
              </div>
            </div>
          </div>
          <div v-if="showCloseButton" class="sheet-close-btn">
            <img
              src="@/assets/images/close-pane.svg"
              alt="close"
              @click="setClose()"
            />
          </div>
        </slot>
        <div class="sheet-content">
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useGesture } from "@vueuse/gesture";
import { useMotionProperties, useMotionTransitions } from "@vueuse/motion";
import { nextTick, onBeforeUnmount, onMounted, ref } from "vue";
import { useLockScroll } from "@/helpers/lockScroll";

const props = defineProps({
  showCloseButton: {
    type: Boolean,
    default: true,
  },
  title: {
    type: String,
    default: "",
  },
  subtitle: {
    type: String,
    default: "",
  },
  openY: {
    type: Number,
    default: 0.1,
  },
  halfY: {
    type: Number,
    default: 0.8,
  },
  defaultState: {
    type: String,
    default: "close",
  },
  barColor: {
    type: String,
    default: "#16192A",
  },
});
onBeforeUnmount(async () => {
  await nextTick();
  const { clearLockScroll } = useLockScroll(
    document.querySelector("html body")
  );

  clearLockScroll();
  window.onresize = null;
});

const sheetRef = ref();
let sheetContent = ref(0);
let state = ref();
let axisY = ref(0);
const DRAG_BAR_HEIGHT = 100;
const BOTTOM_PADDING = 800;

let windowHeight = ref(0);

const { motionProperties } = useMotionProperties(sheetRef);
const { push } = useMotionTransitions(motionProperties);
const spring = {
  stiffness: 550,
  damping: 30,
  restDelta: 0.01,
  restSpeed: 10,
};
const keyFrame = {
  type: "keyframe",
  ease: "linear",
  duration: 0,
  delay: 0,
};
let selectedProp = ref(keyFrame);

onMounted(() => {
  calculateHeight();
});

useGesture(
  {
    onDrag: (ctx) => handleDrag(ctx),
    onDragEnd: (ctx) => handleDragEnd(ctx),
  },
  {
    domTarget: sheetRef,
    drag: {
      filterTaps: true,
      preventWindowScrollY: true,
      useTouch: true,
    },
  }
);

function calculateHeight() {
  nextTick(() => {
    sheetContent.value = sheetRef.value.clientHeight;
    windowHeight.value = window.innerHeight;

    // set sheet to hidden
    axisY.value = windowHeight.value - DRAG_BAR_HEIGHT;
    push("y", axisY.value, motionProperties, keyFrame);
  });
}
function handleDrag(ctx) {
  if (ctx.tap) return;

  const {
    movement: [x, y],
  } = ctx;

  let setY = axisY.value + y;

  push("y", setY, motionProperties, selectedProp.value);
  state.value = "dragging";
}
function handleDragEnd(ctx) {
  if (ctx.tap) return;

  const {
    swipe: [sx, sy],
    movement: [x, y],
  } = ctx;

  //   Handle Swipe

  // swipe down
  if (sy > 0) {
    setClose();
    return;
  }
  //   swipe up
  else if (sy < 0) {
    setOpen();
    return;
  }

  axisY.value = axisY.value + y;

  // Handle Drag

  // drag stop position > 1/3 of the sheet => set to Open
  if (
    windowHeight.value - (axisY.value - DRAG_BAR_HEIGHT) >
    windowHeight.value / 3
  ) {
    setOpen();
    return;
  } else {
    setClose();
    return;
  }
}

async function setOpen() {
  await nextTick();
  const { lockScroll } = useLockScroll(document.querySelector("html body"));
  lockScroll();
  axisY.value = windowHeight.value - sheetContent.value + BOTTOM_PADDING;
  push("y", axisY.value, motionProperties, {
    ...selectedProp.value,
    duration: 100,
  });
  state.value = "open";
}
async function setClose() {
  await nextTick();
  const { unlockScroll } = useLockScroll(document.querySelector("html body"));
  unlockScroll();
  axisY.value = windowHeight.value - DRAG_BAR_HEIGHT;
  push("y", axisY.value, motionProperties, {
    ...selectedProp.value,
    duration: 100,
  });
  state.value = "close";
}
</script>

<style lang="scss" scoped>
.sheet-wrapper[data-open="1"] {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1000;
}

.sheet-overlay-bg {
  transition: all 100ms linear 0ms;
}
.sheet-wrapper[data-open="1"] .sheet-overlay-bg {
  display: block;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    132.34deg,
    rgba(73, 73, 73, 0.2) 14%,
    rgba(73, 73, 73, 0.1) 85.8%
  );
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  z-index: 1000;
}
.modal-sheet {
  touch-action: none;
  background-color: white;
  padding: 0 16px;
  box-shadow: 0px -6px 24px rgb(10 15 45 / 9%);
  border-radius: 15px 15px 0 0;
  position: fixed;
  left: 0;
  right: 0;
  z-index: 1001;

  max-height: 95%;
  margin: 0;
  user-select: none;
  padding-bottom: 800px;
  -webkit-overflow-scrolling: touch;

  button.active {
    background-color: red;
  }
  .bar {
    width: 42px;
    height: 4px;
    border-radius: 14px;
    margin: 6px auto;
    background-color: rgb(22, 25, 42);
    cursor: pointer;
  }

  .st-sticky-header {
    position: -webkit-sticky;
    position: sticky;
    top: 0;
    z-index: 13;
    padding: 16px 0 10px 0;
    background: white;
  }
  .sheet-title {
    font-weight: 600;
    font-size: 20px;
    line-height: 18px;
    color: #191d2f;
  }
  .sheet-subtitle {
    font-weight: 400;
    font-size: 14px;
    line-height: 18px;
    color: #9496ac;
  }
  .sheet-close-btn {
    cursor: pointer;
    z-index: 1001;
    position: absolute;
    right: 16px;
    top: 21px;
  }
  .sheet-content {
    overflow-y: auto;
    max-height: 95vh;
  }
}
</style>
