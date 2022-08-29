<script setup>
import dayjs from 'dayjs'
import { ref, watch } from 'vue'

const value = dayjs(new Date())
const day = ref()

if (dayjs(localStorage.getItem('day'))){
	day.value = value
} else {
	day.value = dayjs(localStorage.getItem('day'))
}

watch(day, (newVal) => {
	if (newVal === null ) {
			day.value = value
		} else {
			const nD = `${newVal.$y}-${newVal.$M + 1}-${newVal.$D}`
			localStorage.setItem('day', newVal)
			localStorage.setItem('dateRequest', nD)
		}
})

</script>

<template>
	<section class="header">
		<h2 class="header_title">
			Актуальные курсы валют на
		</h2>
		<a-date-picker placeholder="Выберите дату" class="header_title__date" v-model:value="day" />
	</section>
</template>

<style scoped>
.header {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(250px, 300px));
	grid-auto-rows: 1fr;
	justify-content: center;
}
.header_title__date{
	margin-bottom: 0;
}
</style>