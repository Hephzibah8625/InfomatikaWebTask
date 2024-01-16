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
const updateKey = ref(0);

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
  setHexagonStyles();


  window.addEventListener('wheel', handleWheel);
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('wheel', handleWheel);
  window.removeEventListener('resize', handleResize);
});


// Functions

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
      newDisplayList.push(listToDisplay.value[i + displayDelta.value]);
    }
    else if (i >= initList.length) {
      newDisplayList.push(listToDisplay.value[listToDisplay.value.length - 1 - (i - initList.length)]);
    }
    else {
      newDisplayList.push(initList[i]);
    }
  }

  listToDisplay.value = newDisplayList;

  calculateCoords();
  setHexagonStyles();
}

const handleWheel = (event) => {
  if (event.deltaY > 0) {
    handleWheelDown();
  }
  else {
    handleWheelUp();
  }
}

const handleWheelUp = () => {
  if (currentIndex.value > 0) {
    currentIndex.value--;
    startIndex.value--;
    endIndex.value--;

    // Добавляем новый
    const isNull = startIndex.value < 0;

    const newElem = isNull ? {} : initList[startIndex.value];

    const styles = {
      position: 'absolute',
      transition: 'all 1s ease',
      height: `${coords.value[0].height}px`,
      width: `${coords.value[0].width}px`,
      top: `${coords.value[0].y}px`,
      left: `${coords.value[0].x}px`,
    };

    if (isNull) {
      styles.display = 'none';
      newElem.isNull = isNull;
      newElem.id = Math.floor(15 + Math.random() * 100);
    }

    newElem.styles = styles;
    newElem.height = coords.value[0].height;
    newElem.coordIdx = 0;

    listToDisplay.value.unshift(newElem);

    // Двигаем хексагоны вверх по координатам

    for (let i = 0; i < listToDisplay.value.length; i++) {

      const coordIdx = listToDisplay.value[i].coordIdx;

      const styles = {
        ...listToDisplay.value[i].styles,
        height: `${coords.value[coordIdx + 1].height}px`,
        width: `${coords.value[coordIdx + 1].width}px`,
        top: `${coords.value[coordIdx + 1].y}px`,
        left: `${coords.value[coordIdx + 1].x}px`,
      };

      listToDisplay.value[i] = {
        ...listToDisplay.value[i],
        height: coords.value[coordIdx + 1].height,
        styles: styles,
        coordIdx: coordIdx + 1,
      };
    }

    // Удаляем самый последний
    listToDisplay.value.pop();
  }
}

const handleWheelDown = () => {
  if (currentIndex.value < initList.length - 1) {
    currentIndex.value++;
    startIndex.value++;
    endIndex.value++;

    // Добавляем новый
    const isNull = endIndex.value >= initList.length;

    const newElem = isNull ? {} : initList[endIndex.value];

    const styles = {
      position: 'absolute',
      transition: 'all 1s ease',
      height: `${coords.value[coords.value.length - 1].height}px`,
      width: `${coords.value[coords.value.length - 1].width}px`,
      top: `${coords.value[coords.value.length - 1].y}px`,
      left: `${coords.value[coords.value.length - 1].x}px`,
    };

    if (isNull) {
      styles.display = 'none';
      newElem.isNull = isNull;
      newElem.id = Math.floor(15 + Math.random() * 100);
    }

    newElem.styles = styles;
    newElem.height = coords.value[coords.value.length - 1].height;
    newElem.coordIdx = coords.value.length - 1;

    listToDisplay.value.push(newElem);

    // Перемещаем хексагоны вниз по координатам

    for (let i = 0; i < listToDisplay.value.length; i++) {

      const coordIdx = listToDisplay.value[i].coordIdx;

      const styles = {
        ...listToDisplay.value[i].styles,
        height: `${coords.value[coordIdx - 1].height}px`,
        width: `${coords.value[coordIdx - 1].width}px`,
        top: `${coords.value[coordIdx - 1].y}px`,
        left: `${coords.value[coordIdx - 1].x}px`,
      };

      listToDisplay.value[i].height = coords.value[coordIdx - 1].height;
      listToDisplay.value[i].styles = styles;
      listToDisplay.value[i].coordIdx = coordIdx - 1;
    }

    updateKey.value++;
    
    
    // Удаляем самый первый
    listToDisplay.value.shift();
  }
}

const onEventClick = (event) => {
  const toIndex = initList.findIndex((i) => i === event.target.id);
  if (toIndex < 0) {
    alert('Error');
    return;
  }
  while (currentIndex.value !== toIndex) {
    if (currentIndex.value < toIndex) {
      handleWheelDown();
    }
    else {
      handleWheelUp();
    }
  }
}

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
      x: coords.value[i + 1].x - (100 + iWidth),
      y: coords.value[i + 1].y + coords.value[i + 1].height - iHeight / 2,
      height: iHeight,
      width: iWidth,
    };
  }

  for (let i = centerIdx + 1; i < coords.value.length; i++) {
    const iHeight = coords.value[i - 1].height * 0.7;
    const iWidth = coords.value[i - 1].width * 0.7;
    coords.value[i] = {
      x: coords.value[i - 1].x + (100 + coords.value[i - 1].width),
      y: coords.value[i - 1].y - iHeight / 2, 
      height: iHeight,
      width: iWidth,
    };
  }
};

const setHexagonStyles = () => {
  for (let i = 0; i < listToDisplay.value.length; i++) {

    const styles = {
      position: 'absolute',
      transition: 'all 1s ease',
      height: `${coords.value[i + 1].height}px`,
      width: `${coords.value[i + 1].width}px`,
      top: `${coords.value[i + 1].y}px`,
      left: `${coords.value[i + 1].x}px`,
    };

    if (listToDisplay.value[i].isNull) {
      styles.display = 'none';
    };

    listToDisplay.value[i].styles = styles;
    listToDisplay.value[i].height = coords.value[i + 1].height;

    listToDisplay.value[i].coordIdx = i + 1;
  }
};

const getPositionStyle = (top, left, width, height) => {
  return {
    top: `${top}px`,
    left: `${left}px`,
    width: `${width}px`,
    height: `${height}px`,
  }
}

const getCircleStyle = (idx) => {
  console.log(idx);
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
}

</script>

<template>
  <div class="wrapper" ref="wrapper">
    <div class="navbar">
      <div><a href="#">БИЛЕТЫ И АБОНИМЕНТЫ</a></div>

      <div class="navbar__menu">
        <div><a href="#">Как купить?</a></div>
        <div><a href="#">Правила</a></div>
        <div><a href="#">Возврат билетов</a></div>
      </div>

      <div class="navbar__sign-in">
        <a href="#">
          <span class="material-symbols-outlined">login</span>Войти<span class="material-symbols-outlined">menu</span>
        </a>
      </div>
    </div>

    
    <div
      class="enemy-line"
      :style="getPositionStyle(
        height / 2 - initHeight / 2,
        0,
        width / 2 - initWidth * 2,
        initHeight / 2
      )"
    >
      <div>{{ initList[currentIndex].left_team }}</div>
    </div>
    <div
      class="enemy-line"
      :style="getPositionStyle(
        height / 2,
        width / 2 + initWidth * 2,
        width / 2 - initWidth * 2,
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
      @click="onEventClick"
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
}

.navbar__sign-in a {
  display: flex;
  align-items: center;
  gap: 1vw;
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
  font-size: calc(var(--index) * 1.2);
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
}

.circle {
  position: absolute;
  border: 1px solid lightgray;
  border-radius: 100%;
  z-index: -1000;
}

</style>
