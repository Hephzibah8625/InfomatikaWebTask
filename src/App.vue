<script setup>
import { onBeforeMount, onBeforeUnmount, onMounted, ref } from 'vue';
import Hexagon from './components/Hexagon.vue';

const initList = [
  { id: 0, timestamp: 1727722800000, place: 'Стадион 1', left_team: 'Соперник №1', right_team: 'Соперник №2', },
  { id: 1, timestamp: 1721147400000, place: 'Стадион 2', left_team: 'Соперник №1', right_team: 'Соперник №3', },
  { id: 2, timestamp: 1719428400000, place: 'Стадион 3', left_team: 'Соперник №1', right_team: 'Соперник №2', },
  { id: 3, timestamp: 1687011300000, place: 'Стадион 4', left_team: 'Соперник №2', right_team: 'Соперник №3', },
  { id: 4, timestamp: 1717102800000, place: 'Стадион 5', left_team: 'Соперник №3', right_team: 'Соперник №1', },
  { id: 5, timestamp: 1712232000000, place: 'Стадион 6', left_team: 'Соперник №3', right_team: 'Соперник №4', },
  { id: 6, timestamp: 1710578700000, place: 'Стадион 6', left_team: 'Соперник №3', right_team: 'Соперник №5', },
];

const currentIndex = ref(Math.floor(initList.length % 2 === 0 ? initList.length / 2 - 1 : initList.length / 2));
const listToDisplay = ref([]);
const coords = ref([]);
const width = ref(window.innerWidth);
const height = ref(window.innerHeight);
const wrapper = ref(null);
const initWidth = ref(width.value > 1000 ? width.value / 10 : width.value / 5);
const initHeight = ref(initWidth.value / 0.645);
const objectsToDisplay = ref(width.value > 1000 ? 5 : 3);
const displayDelta = ref(Math.floor(objectsToDisplay.value / 2));
const startIndex = ref(currentIndex.value - displayDelta.value);
const endIndex = ref(currentIndex.value + displayDelta.value);
const touchY = ref(0);

// Lifecycle Hooks

onBeforeMount(() => {
  calculateCoords();
});

onMounted(() => {
  for (let i = startIndex.value; i <= endIndex.value; i++) {
    if (i < 0 || i >= initList.length) continue;

    listToDisplay.value = [ ...listToDisplay.value, initList[i] ];
  }

  calculateCoords();
  setHexagonsStyles();


  window.addEventListener('wheel', handleWheel);
  window.addEventListener('resize', handleResize);
  window.addEventListener('touchstart', handleTouchStart);
  window.addEventListener('touchend', handleTouchEnd);
});

onBeforeUnmount(() => {
  window.removeEventListener('wheel', handleWheel);
  window.removeEventListener('resize', handleResize);
  window.addEventListener('touchstart', handleTouchStart);
  window.addEventListener('touchend', handleTouchEnd);
});


// Functions

const handleTouchStart = (event) => {
  touchY.value = event.changedTouches[0].pageY;
}

const handleTouchEnd = (event) => {
  const offsetY = touchY.value - event.changedTouches[0].pageY;
  if (offsetY > 0) {
    handleWheelUp();
  }
  else {
    handleWheelDown();
  }
}

const handleResize = () => {
  width.value = window.innerWidth;
  height.value = window.innerHeight;

  initWidth.value = width.value > 1000 ? width.value / 10 : width.value / 5;
  initHeight.value = initWidth.value / 0.645;

  objectsToDisplay.value = width.value > 1000 ? 5 : 3;
  displayDelta.value = Math.floor(objectsToDisplay.value / 2);

  startIndex.value = currentIndex.value - displayDelta.value;
  endIndex.value = currentIndex.value + displayDelta.value;

  const newDisplayList = [];

  for (let i = startIndex.value; i <= endIndex.value; i++) {
    if (i < 0) {
      const newElem = {};
      setHexagonParams(newElem, true, i + displayDelta.value);
      newElem.isNull = true;
      newElem.id = getNewUniqeId();
      newDisplayList.push(newElem);
    }
    else if (i >= initList.length) {
      const newElem = {};
      setHexagonParams(newElem, true, coords.value.length - (i - initList.length) - 1);
      newElem.isNull = true;
      newElem.id = getNewUniqeId();
      newDisplayList.push(newElem);
    }
    else {
      newDisplayList.push(initList[i]);
    }
  }

  listToDisplay.value = newDisplayList;

  calculateCoords();
  setHexagonsStyles();
};

const handleWheel = (event) => {
  if (event.deltaY > 0) {
    handleWheelDown();
  }
  else {
    handleWheelUp();
  }
};

const handleWheelUp = () => {
  if (currentIndex.value > 0) {
    currentIndex.value--;
    startIndex.value--;
    endIndex.value--;

    // Добавляем новый
    const isNull = startIndex.value < 0;

    const newElem = isNull ? {} : initList[startIndex.value];

    setHexagonParams(newElem, isNull, 0);

    if (isNull) {
      newElem.isNull = isNull;
      newElem.id = getNewUniqeId();
    }

    listToDisplay.value.unshift(newElem);

    // Двигаем хексагоны вверх по координатам

    for (let i = 0; i < listToDisplay.value.length; i++) {

      const coordIdx = listToDisplay.value[i].coordIdx;

      setHexagonParams(listToDisplay.value[i], listToDisplay.value[i].isNull, coordIdx + 1);
    }

    // Удаляем самый последний
    listToDisplay.value.pop();
  }
};

const handleWheelDown = () => {
  if (currentIndex.value < initList.length - 1) {
    currentIndex.value++;
    startIndex.value++;
    endIndex.value++;

    // Добавляем новый
    const isNull = endIndex.value >= initList.length;

    const newElem = isNull ? {} : initList[endIndex.value];

    setHexagonParams(newElem, isNull, coords.value.length - 1);

    if (isNull) {
      newElem.isNull = isNull;
      newElem.id = getNewUniqeId();
    }

    listToDisplay.value.push(newElem);

    // Перемещаем хексагоны вниз по координатам

    for (let i = 0; i < listToDisplay.value.length; i++) {

      const coordIdx = listToDisplay.value[i].coordIdx;

      setHexagonParams(listToDisplay.value[i], listToDisplay.value[i].isNull, coordIdx - 1);
    }
    
    // Удаляем самый первый
    listToDisplay.value.shift();
  }
};

const onEventClick = (index) => {
  let toIndex = index - Math.floor(listToDisplay.value.length / 2);
  while (toIndex !== 0) {
    if (toIndex < 0) {
      handleWheelUp();
      toIndex++;
    }
    else {
      handleWheelDown();
      toIndex--;
    }
  }
};

const calculateCoords = () => {
  const centerX = width.value / 2;
  const centerY = height.value / 2;
  coords.value = [];
  const size = objectsToDisplay.value + 2;
  const centerIdx = Math.floor(size % 2 === 0 ? size / 2 - 1 : size / 2);

  for (let i = 0; i < size; i++) coords.value.push({});

  coords.value[centerIdx] = {
    x: centerX - initWidth.value / 2,
    y: centerY - initHeight.value / 2,
    height: initHeight.value,
    width: initWidth.value,
  };

  for (let i = centerIdx - 1; i >= 0; i--) {
    const iHeight = coords.value[i + 1].height * 0.7;
    const iWidth = coords.value[i + 1].width * 0.7;

    coords.value[i] = {
      x: coords.value[i + 1].x - (width.value * 0.05 + iWidth),
      y: coords.value[i + 1].y + coords.value[i + 1].height - iHeight / 2,
      height: iHeight,
      width: iWidth,
    };
  }

  for (let i = centerIdx + 1; i < coords.value.length; i++) {
    const iHeight = coords.value[i - 1].height * 0.7;
    const iWidth = coords.value[i - 1].width * 0.7;
    coords.value[i] = {
      x: coords.value[i - 1].x + (width.value * 0.05 + coords.value[i - 1].width),
      y: coords.value[i - 1].y - iHeight / 2, 
      height: iHeight,
      width: iWidth,
    };
  }
};

const setHexagonsStyles = () => {
  for (let i = 0; i < listToDisplay.value.length; i++) {
    setHexagonParams(listToDisplay.value[i], listToDisplay.value[i].isNull, i + 1);
  }
};

const setHexagonParams = (obj, isNull, coordIdx) => {
  const styles = {
    ...obj.styles,
    position: 'absolute',
    transition: 'all 1s ease',
    height: `${coords.value[coordIdx].height}px`,
    width: `${coords.value[coordIdx].width}px`,
    top: `${coords.value[coordIdx].y}px`,
    left: `${coords.value[coordIdx].x}px`,
  };

  if (isNull) {
    styles.display = 'none';
  }

  obj.styles = styles;
  obj.height = coords.value[coordIdx].height;
  obj.coordIdx = coordIdx;
}

const getPositionStyle = (top, left, width, height) => {
  return {
    top: `${top}px`,
    left: `${left}px`,
    width: `${width}px`,
    height: `${height}px`,
  }
};

const getCircleStyle = (idx) => {
  const middleIdx = Math.floor(coords.value.length / 2);

  const dH = coords.value[middleIdx].y - coords.value[middleIdx + idx].y + coords.value[middleIdx].height / 2;
  const dW = coords.value[middleIdx + idx].x - coords.value[middleIdx].x - coords.value[middleIdx].width / 2;

  const radius = Math.sqrt(dH * dH + dW * dW);

  return getPositionStyle(
    height.value / 2 - radius,
    width.value / 2 - radius,
    radius * 2,
    radius * 2
  );
};

const getNewUniqeId = () => {
  let id = Math.floor(15 + Math.random() * 100);
  while (initList.findIndex((i) => i.id === id) !== -1) {
    id = Math.floor(15 + Math.random() * 100);
  }
  return id;
}

</script>

<template>
  <div class="wrapper" ref="wrapper">
    <div class="navbar">
      <div><a href="#">БИЛЕТЫ И АБОНЕМЕНТЫ</a></div>

      <div class="navbar__menu">
        <div><a href="#">Как купить?</a></div>
        <div><a href="#">Правила</a></div>
        <div><a href="#">Возврат билетов</a></div>
      </div>

      <div class="navbar__sign-in">
        <span class="material-symbols-outlined">login</span>
        <a href="#">Войти</a>
        <span class="material-symbols-outlined">menu</span>
      </div>
    </div>

    
    <div
      class="enemy-line"
      :style="getPositionStyle(
        height / 2 - initHeight / 2,
        0,
        width / 2 - initWidth * 1.4,
        initHeight / 2
      )"
    >
      <div>{{ initList[currentIndex].left_team }}</div>
    </div>
    <div
      class="enemy-line"
      :style="getPositionStyle(
        height / 2,
        width / 2 + initWidth * 1.4,
        width / 2 - initWidth * 1.4,
        initHeight / 2
      )"
    >
      <div>{{  initList[currentIndex].right_team }}</div>
    </div>

    <div
      class="circle"
      v-for="i in displayDelta"
      :style="getCircleStyle(i)"
    />

    <Hexagon
      v-for="(hexagonData, hexagonIdx) of listToDisplay"
      :key="hexagonData.id"
      :hex-data="hexagonData"
      :height="hexagonData.height"
      :style="hexagonData.styles"
      :on-center="hexagonIdx === Math.floor(listToDisplay.length / 2)"
      @click="onEventClick(hexagonIdx)"
    />
  </div>
</template>

<style scoped>
.wrapper {
  width: 100vw;
  height: 100vh;
  position: relative;
}

.navbar {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  color: white;
  padding: calc(1vh + 1vw);
  display: grid;
  grid-template-columns: 1fr auto 1fr;
}

.navbar__sign-in {
  justify-self: end;
  display: flex;
  align-items: center;
  gap: 1vw;
}

.navbar__sign-in span {
  cursor: pointer;
}

.navbar__menu {
  display: flex;
  gap: 5vw;
}

a, span {
  font-size: calc(var(--index) * 0.8);
}

a {
  color: white;
  position: relative;
}

a:before {
  content: '';
  position: absolute;
  width: 100%;
  height: calc(var(--index) * 0.1);
  border-radius: calc(var(--index) * 0.1);
  background-color: white;
  bottom: 0;
  left: 0;
  transform-origin: right;
  transform: scaleX(0);
  transition: transform .3s ease-in-out;
}

a:hover::before {
  transform-origin: left;
  transform: scaleX(1);
}

.enemy-line {
  position: absolute;
  background-color: white;
  color: black;
  font-size: calc(var(--index) * 1.2);
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
}

.circle {
  position: absolute;
  border: 1px solid darkgray;
  border-radius: 100%;
  z-index: -1000;
}

</style>
