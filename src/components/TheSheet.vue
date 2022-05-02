<template>
  <div class="sheet-wrapper" v-motion="'demo'" ref="sheetRef" :style="getStyle">
    <div class="bar"></div>
    <h2>Sheet Title</h2>
    <div class="sheet-content">
      Lorem ipsum, dolor sit amet consectetur adipisicing elit. Totam doloribus
      expedita quod nesciunt, eius voluptas? Dolorum amet odio aliquid rem,
      itaque quae, suscipit architecto aspernatur dignissimos minus, possimus
      blanditiis ea?
      <button>Click</button>
      <div style="margin: 30px 0" v-for="i in 10" :key="i">
        Lorem ipsum dolor, sit amet consectetur adipisicing elit. Obcaecati
        aliquam molestiae ad delectus maiores libero, suscipit nam ullam atque,
        temporibus, a ab harum cupiditate minus velit! Itaque explicabo sit
        repellendus?
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from "@vue/reactivity";
import { useGesture } from "@vueuse/gesture";
import { useMotionProperties, useSpring } from "@vueuse/motion";
import { nextTick, onMounted, ref } from "vue";

const sheetRef = ref();
let sheetContent = ref(0);
let state = ref();
let axisY = ref(0);
const DRAG_BAR_HEIGHT = 20;
const BOTTOM_PADDING = 500;

let windowHeight = ref(0);
const { motionProperties } = useMotionProperties(sheetRef);
const { set } = useSpring(
  motionProperties
  // , {
  //   damping: 50,
  //   stiffness: 220,
  // }
);

onMounted(() => {
  calculateHeight();
});

useGesture(
  {
    onDrag: (ctx) => handleDrag(ctx),
    onDragEnd: (ctx) => handleDragEnd(ctx),

    // onHover: ({ hovering }) =>
    //   !hovering && set({ rotateX: 0, rotateY: 0, scale: 1 }),
    // onMove: ({ xy: [px, py], dragging }) =>
    //   !dragging &&
    //   set({
    //     rotateX: calcX(py, motionProperties.y),
    //     rotateY: calcY(px, motionProperties.x),
    //     scale: 1.2,
    //   }),
    // onPinch: ({ offset: [d, a] }) => {
    //   console.log(d, a);
    // },
    // onWheel: ({ movement: [x, y] }) =>
    //   boxSet({
    //     y,
    //     x,
    //   }),
    // onWheelEnd: () =>
    //   boxSet({
    //     y: 0,
    //     x: 0,
    //   }),
  },
  {
    domTarget: sheetRef,
    drag: {
      filterTaps: true,
      preventWindowScrollY: true,
      useTouch: true,
      delay: 5000,
    },
  }
);

// const getStyle = computed(() => {
//   console.log("sheetContent.value: ", sheetContent);
//   return {
//     height: `${sheetContent.value + BOTTOM_PADDING}px`,
//   };
// });

function calculateHeight() {
  nextTick(() => {
    sheetContent.value = sheetRef.value.clientHeight;
    windowHeight.value = window.innerHeight;

    // set sheet to hidden
    axisY.value = windowHeight.value - DRAG_BAR_HEIGHT;

    set({
      x: 0,
      y: axisY.value,
      //    rotateX: 0,
      //     rotateY: 0,
      //      scale: 1
    });
  });
}
function handleDrag(ctx) {
  const {
    movement: [x, y],
  } = ctx;
  let setY = axisY.value + y;

  set({
    x: 0,
    y: setY,
  });

  state.value = "dragging";
}
function handleDragEnd(ctx) {
  //   Handle Swipe
  const {
    swipe: [sx, sy],
  } = ctx;

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
  const {
    movement: [x, y],
  } = ctx;

  // drag stop position > 1/3 of the sheet => set to Open
  if (windowHeight.value - axisY.value > sheetContent.value / 3) {
    setOpen();
    return;
  } else {
    setClose();
    return;
  }
}

function setOpen() {
  axisY.value = windowHeight.value - sheetContent.value;
  set({
    x: 0,
    y: axisY.value,
  });
}
function setClose() {
  axisY.value = windowHeight.value - DRAG_BAR_HEIGHT;

  set({
    x: 0,
    y: axisY.value,
  });
}
</script>

<style lang="scss" scoped>
.sheet-wrapper {
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
