<script setup>
import { onMounted, ref, watch, reactive } from "vue";
import axios from "axios";

import Header from "./components/Header.vue";
import CardList from "./components/CardList.vue";
import Drawer from "./components/Drawer.vue";

const items = ref([]);
// для фильтрации и поиска
const filters = reactive({
	sortBy: "title",
	searchQuery: "",
});

// сортировка
// при изменении значения select - меняется sortBy
const onChangeSelect = (event) => {
	filters.sortBy = event.target.value;
};
// поиск
const onChangeSearchInput = (event) => {
	filters.searchQuery = event.target.value;
};

// запрос на бэк
const fetchItems = async () => {
	try {
		// параметры сортировки и поиска
		const params = {
			sortBy: filters.sortBy,
			// searchQuery: filters.searchQuery,
		};
		if (filters.searchQuery) {
			params.title = `*${filters.searchQuery}*`;
		}
		const { data } = await axios.get(
			"https://280b3c98e5c903e5.mokky.dev/items",
			{
				params,
			}
		);
		// используя ref нужно всегда вытаскивать value
		items.value = data;
	} catch (error) {
		console.log(error);
	}
};

// функция для запроса с бэка
// при первом рендере приложения! с помощью хука обращаемся к бэку
onMounted(fetchItems);
// изменение сортировки по фильтру
// watch - следит за изменениями sortBy
watch(filters, fetchItems);
</script>

<template>
	<!-- <Drawer /> -->
	<div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
		<!-- <h1>Sergei</h1> -->
		<Header />
		<div class="p-10">
			<div class="flex justify-between items-center">
				<h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

				<div class="flex gap-4">
					<select
						@change="onChangeSelect"
						name=""
						id=""
						class="px-3 py-2 border rounded-md outline-none"
					>
						<option value="name">По названию</option>
						<option value="price">По цене (дешевые)</option>
						<option value="-price">По цене (дорогие)</option>
					</select>

					<div class="relative">
						<img
							class="absolute left-3 top-3"
							src="/search.svg"
							alt="search"
						/>
						<input
							@input="onChangeSearchInput"
							type="text"
							placeholder="Поиск..."
							class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
						/>
					</div>
				</div>
			</div>
			<div class="mt-10">
				<CardList :items="items" />
			</div>
		</div>
	</div>
</template>

<style scoped></style>
