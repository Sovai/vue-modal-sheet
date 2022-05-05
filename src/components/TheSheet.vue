<template>
  <button @click="selectedProp === spring">Spring</button>
  <button @click="selectedProp = keyFrame">Keyframe</button>

  <div class="sheet-wrapper" v-motion="'demo'" ref="sheetRef">
    <div class="bar"></div>
    <h2>Sheet Title</h2>
    <div class="sheet-content">
      Lorem ipsum, dolor sit amet consectetur adipisicing elit. Totam doloribus
      expedita quod nesciunt, eius voluptas? Dolorum amet odio aliquid rem,
      itaque quae, suscipit architecto aspernatur dignissimos minus, possimus
      blanditiis ea?
      <button>Click</button>
      <div style="margin: 30px 0" v-for="i in 2" :key="i">
        Lorem ipsum dolor, sit amet consectetur adipisicing elit. Obcaecati
        aliquam molestiae ad delectus maiores libero, suscipit nam ullam atque,
        temporibus, a ab harum cupiditate minus velit! Itaque explicabo sit
        repellendus?
      </div>
    </div>
  </div>
</template>

<script setup>
import { useGesture } from "@vueuse/gesture";
import {
  useMotionControls,
  useMotionProperties,
  useMotion,
  useSpring,
  useMotionTransitions,
} from "@vueuse/motion";
import { nextTick, onMounted, ref } from "vue";
// const props = defineProps({
//   showCloseButton: {
//     type: Boolean,
//     default: true,
//   },
//   title: {
//     type: String,
//     default: "",
//   },
//   subtitle: {
//     type: String,
//     default: "",
//   },
//   openY: {
//     type: Number,
//     default: 0.1,
//   },
//   halfY: {
//     type: Number,
//     default: 0.8,
//   },
//   defaultState: {
//     type: String,
//     default: "close",
//   },
//   barColor: {
//     type: String,
//     default: "#16192A",
//   },
// });

const sheetRef = ref();
let sheetContent = ref(0);
let state = ref();
let axisY = ref(0);
const DRAG_BAR_HEIGHT = 100;
// const BOTTOM_PADDING = 500;

let windowHeight = ref(0);

const { motionProperties } = useMotionProperties(sheetRef);
const { push, stop } = useMotionTransitions(motionProperties);
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
console.log("motionProp: ", { motionProperties });

const { set } = useSpring(motionProperties);
// const { set } = useMotionControls(motionProperties);

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
    push("y", axisY.value, motionProperties, {
      type: "spring",
      bounce: 0,
    });

    // set({
    //   x: 0,
    //   y: axisY.value,
    // });
  });
}
function handleDrag(ctx) {
  const {
    movement: [x, y],
  } = ctx;
  let setY = axisY.value + y;
  console.log({
    y,
    axisY: axisY.value,
    setY,
    sheetContent: sheetContent.value,
    windowHeight: windowHeight.value,
  });
  // if (setY < sheetContent.value) return;
  push("y", setY, motionProperties, selectedProp.value);

  // set({
  //   x: 0,
  //   y: setY,
  // });

  state.value = "dragging";
}
function handleDragEnd(ctx) {
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
  if (windowHeight.value - axisY.value > windowHeight.value / 3) {
    setOpen();
    return;
  } else {
    setClose();
    return;
  }
}

function setOpen() {
  axisY.value = windowHeight.value - sheetContent.value;
  push("y", axisY.value, motionProperties, {
    ...selectedProp.value,
    duration: 100,
  });

  // set({
  //   x: 0,
  //   y: axisY.value,
  // });
}
function setClose() {
  axisY.value = windowHeight.value - DRAG_BAR_HEIGHT;
  push("y", axisY.value, motionProperties, {
    ...selectedProp.value,
    duration: 100,
  });

  // set({
  //   x: 0,
  //   y: axisY.value,
  // });
}
</script>

<style lang="scss" scoped>
.sheet-wrapper {
  touch-action: none;

  background-color: white;
  padding: 0 16px;
  box-shadow: 0px -6px 24px rgb(10 15 45 / 9%);
  border-radius: 15px 15px 0 0;

  position: absolute;
  //   bottom: -300px;
  left: 0;
  right: 0;
  //   width: 100%;
  margin: 0;
  user-select: none;

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
  h2 {
    font-size: 1.5rem;
    font-weight: bold;
  }
}
</style>
