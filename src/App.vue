<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import Header from './Header.vue'

const currencysName = ref([])
const dateReq = ref(localStorage.getItem('dateRequest'))
const arrCurrencys = ref([])
const input_content = ref(['USD','EUR','RUB'])
const flag = ref(true)
const url = `https://www.nbrb.by/api/exrates/rates?ondate=${dateReq.value}&periodicity=0`;
const errors = [];

const handleChange = value => input_content.value = value

async function localName() {
	await fetch(url)
		.then((response) => response.json())
		.then((data) => {
			const dlSet = data.map(element => element['Cur_Abbreviation'])
			currencysName.value = [...new Set(dlSet)]
		})
		.catch((e) => {
			errors.push(e);
			if (errors) console.log(`errors ${errors.length}`);
		});
}
localName()

async function localDataGet() {
	await fetch(`https://www.nbrb.by/api/exrates/rates?ondate=${dateReq.value}&periodicity=0`)
		.then((response) => response.json())
		.then((data) => {
			let newArr = data.filter(el => input_content.value.filter(it => it == el.Cur_Abbreviation) == el.Cur_Abbreviation)
			localStorage.setItem('arrCurrencys', JSON.stringify(newArr))
			arrCurrencys.value = newArr
		})
		.catch((e) => {
			errors.push(e);
			if (errors) console.log(`errors ${errors.length}`);
		});
}

const addCourse = () => {
	if (!input_content.value.length) {
		return
	}
	dateReq.value = localStorage.getItem('dateRequest')
	localDataGet()
}

const removeTodo = (el) => {
	input_content.value = input_content.value.filter((t) => t !== el.Cur_Abbreviation)
	arrCurrencys.value = arrCurrencys.value.filter(t => t !== el)
}
watch(input_content, () => {
	localStorage.setItem('input_content', Object.values(input_content.value))
})


watch(arrCurrencys, () => {
	if (arrCurrencys.value.length===0) {
		flag.value = false
	} else{
		flag.value = true
	}
})

if (!localStorage.getItem('input_content')) {
	localStorage.setItem('input_content', input_content.value)
}

onMounted(() => {
	input_content.value = localStorage.getItem('input_content').split(',')
	arrCurrencys.value = JSON.parse(localStorage.getItem('arrCurrencys')) || []
})

</script>

<template>
	<main class="app">

		<Header />

		<section class="create-course">
			<form id="new-course-form" @submit.prevent="addCourse">
				<h4>Достаточно выбрать интересующую дату и валюту в специальном поле, и нажать кнопку 'Узнать'. </h4>

				<a-select v-model:value="input_content" class="input_course" mode="multiple" style="width: 100%"
					placeholder="USD" :options="[...currencysName].map((key) => ({ value: key }))" @change="handleChange">
				</a-select>

				<input type="submit" value='Узнать...' />
			</form>
		</section>

		<section class="course-list" v-if="flag">
			<div class="course-item" v-for="el of arrCurrencys" key="el">
				<div class="course-item__title">
					<p>{{ el.Cur_Scale }} {{ el.Cur_Name }} ({{ el.Cur_Abbreviation }}) </p>
					<a-button class="course-item_del-btn" style="margin-bottom: 8px" @click="removeTodo(el)">Delete</a-button>
				</div>
				<hr />

				<div class="course-item__body">
					<p>1 BYN / {{ el.Cur_OfficialRate }} {{ el.Cur_Abbreviation }} </p>
				</div>

				<div class="course-item__footer">
					<p>Акутуальный курс на {{ el.Date.slice(-19, -9) }}</p>
				</div>
			</div>
		</section>

	</main>
</template>

<style scoped>
.input_course {
	display: block;
	width: 100%;
	font-size: 1.125rem;
	color: var(--dark);
	background-color: #FFF;
	border-radius: 0.5rem;
	box-shadow: var(--shadow);
	margin-bottom: 1.5rem;
}

.create-course input[type="submit"] {
	display: block;
	width: 100%;
	font-size: 1.125rem;
	padding: 1rem 1.5rem;
	color: #FFF;
	background-color: var(--primary);
	border-radius: 0.5rem;
	box-shadow: var(--personal-glow);
	cursor: pointer;
	transition: 0.2s ease-in-out;
}

.create-course input[type="submit"]:hover {
	opacity: 0.75;
}

.course-list {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
	grid-auto-rows: 1fr;
	grid-gap: 1rem;
	background-color: #FFF;
	padding: 1rem;
	margin: 1rem;
	border-radius: 0.5rem;
	box-shadow: var(--shadow);
}

.course-item {
	display: block;
	background-color: var(--light);
}
.course-item__title,
.course-item__footer {
	display: grid;
	align-items: center;
	grid-auto-flow: column;
	gap: 30px;
	padding: 0.8rem;
	margin-bottom: 0;
	padding-bottom: 0;
	color: #00000073;
}

.course-item__body {
	display: grid;
	justify-content: center;
	grid-auto-flow: column;
	padding: 1rem;
	color: #000000d9;
	font-size: 1.5rem;
}
.course-item__footer{
	display: grid;
	grid-auto-flow: column;
	padding: 0.5rem;
}
.course-item__footer p{
	align-self: end;
	justify-self: end;
	padding-right: 0.7rem;
	}
</style>