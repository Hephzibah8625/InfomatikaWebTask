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

const calculateOptionFont = () => {
	return {
		transition: 'all 1s ease',
		fontSize: `${props.height * 0.15}px`,
	}
}

const calculatePlaceStyle = () => {
	return {
		transition: 'all 1s ease',
		fontSize: `${props.height * 0.09}px`,
		marginBottom: `${props.height * 0.1}px`,
	}
}

const calculateTimeStyle = () => {
	return {
		transition: 'all 1s ease',
		fontSize: `${props.height * 0.09}px`,
		marginBottom: `${props.height * 0.04}px`,
	}
}

const calculateDateStyle = () => {
	return {
		transition: 'all 1s ease',
		fontSize: `${props.height * 0.12}px`,
		fontWeight: 'bold',
		marginBottom: `${props.height * 0.03}px`,
		textTransform: 'uppercase',
	}
}

const calculateButtonStyle = () => {
	return {
		fontSize: `${props.height * 0.08}px`,
		padding: `${props.height * 0.04}px`,
		border: '1px solid black',
		borderRadius: `${props.height * 0.04}px`,
	}
}

const getTime = (ts) => `${new Date(ts).getUTCHours()}:${new Date(ts).getUTCMinutes()}`;
const getDay = (ts) => new Date(ts).getUTCDate();
const getMonth = (ts) => months[ new Date(ts).getUTCMonth() ];
</script>

<template>
  <div class="hexagon-base" :style="calculateStyle()">
    <div class="hexagon" :style="{ ...calculateStyle(), 'border-radius': `${props.height / 10}px` }" />
    <div v-if="onCenter" class="hexagon-data">
			<div :style="calculatePlaceStyle()">{{ hexData.place }}</div>
			<div :style="calculateDateStyle()">{{ getDay(hexData.timestamp) }} {{ getMonth(hexData.timestamp) }}</div>
			<div :style="calculateTimeStyle()">{{ getTime(hexData.timestamp) }}</div>
			<a :style="calculateButtonStyle()">Купить билет</a>
    </div>
		<div v-else class="hexagon-data" :style="calculateOptionFont()">
			<div>{{ getDay(hexData.timestamp) }}</div>
			<div>{{ getMonth(hexData.timestamp) }}</div>
		</div>
  </div>
</template>

<style scoped>

.hexagon-base {
	transition: all 1s ease;
}

.hexagon {
  width: inherit;
  height: inherit;
  background-color: white;
  border-radius: 30px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 5px 5px -5px black;
}

.hexagon:before, .hexagon:after {
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

.hexagon:after {
  transform: rotate(60deg);
}

.hexagon:before {
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

</style>
