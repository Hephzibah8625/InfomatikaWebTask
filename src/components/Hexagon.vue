<script setup>
const months = [
  'января', 'февраля', 'марта', 'апреля', 'мая', 'июня',
  'июля', 'августа', 'сентября', 'октября', 'ноября', 'декабря'
];

const props = defineProps({
  onCenter: Boolean,
  hexData: Object,
  height: Number,
})

const calculateStyle = () => {
  return {
		transition: 'all 1s ease',
    height: `${props.height}px`,
    width: `${props.height * 0.645}px`,
  }
}

const getTime = (ts) => {
  const hours = new Date(ts).getUTCHours();
  const minutes = new Date(ts).getUTCMinutes();
  return `${hours > 9 ? hours : '0' + hours}:${minutes > 9 ? minutes : '0' + minutes}`;
};
const getDay = (ts) => new Date(ts).getUTCDate();
const getMonth = (ts) => months[ new Date(ts).getUTCMonth() ];
</script>

<template>
  <div class="hexagon" :style="calculateStyle()">
    <div class="hexagon-base" :style="{ ...calculateStyle(), 'border-radius': `${props.height / 10}px` }" />
    <div v-if="onCenter" class="hexagon-data">
			<div class="hexagon-data__place">{{ hexData.place }}</div>
			<div class="hexagon-data__centered-date">{{ getDay(hexData.timestamp) }} {{ getMonth(hexData.timestamp) }}</div>
			<div class="hexagon-data__time">{{ getTime(hexData.timestamp) }}</div>
			<a class="hexagon-data__button" href="#" @click.prevent="">Купить билет</a>
    </div>
		<div v-else class="hexagon-data hexagon-data__option">
			<div>{{ getDay(hexData.timestamp) }}</div>
			<div>{{ getMonth(hexData.timestamp) }}</div>
		</div>
  </div>
</template>

<style scoped>

.hexagon {
	transition: all 1s ease;
  animation: append-animate 0.5s linear;
}

.hexagon-base {
  width: inherit;
  height: inherit;
  background-color: white;
  border-radius: 30px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.hexagon-base:before, .hexagon-base:after {
  content: '';
  display: block;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  width: inherit;
  height: inherit;
  background-color: inherit;
  border-radius: inherit;
}

.hexagon-base:after {
  transform: rotate(60deg);
}

.hexagon-base:before {
  transform: rotate(-60deg);
}

.hexagon-data {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: black;
  display: flex;
  flex-direction: column;
  align-items: center;
	justify-content: center;
}

.hexagon-data__place {
  font-size: calc(var(--index) * 0.7);
  margin-bottom: calc(var(--index) * 0.6);
}

.hexagon-data__centered-date {
  font-size: calc(var(--index) * 1.1);
  font-weight: bold;
  text-transform: uppercase;
  margin-bottom: calc(var(--index) * 0.1);
}

.hexagon-data__time {
  font-size: calc(var(--index) * 0.7);
  margin-bottom: calc(var(--index) * 0.3);
}
.hexagon-data__button {
  color: black;
  font-size: calc(var(--index) * 0.6);
  padding: calc(var(--index) * 0.3);
  border: 1px solid black;
  border-radius: calc(var(--index) * 0.3);
  transition: all 0.5s ease;
}

.hexagon-data__button:hover {
  transform: scale(1.1, 1.1);
}

.hexagon-data__option {
  font-size: calc(var(--index) * 0.8);
}


@keyframes append-animate {
	from {
		transform: scale(0);
		opacity: 0;
	}
	to {
		transform: scale(1);
		opacity: 1;	
	}
}

</style>
