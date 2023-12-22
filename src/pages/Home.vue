<script setup>
import CardList from "../components/CardList.vue";
import axios from "axios";
import debounce from "lodash.debounce";
import { reactive, inject, watch, ref, onMounted } from "vue";

const { cart, addToCart, removeFromCart } = inject("cart");
const items = ref([]);

const filters = reactive({
	sortBy: "title",
	searchQuery: "",
});

const onClickAddPlus = (item) => {
	// если нет в корзине
	if (!item.isAdded) {
		addToCart(item);
	} else {
		// если есть в корзине
		removeFromCart(item);
	}
};
// сортировка
// при изменении значения select - меняется sortBy
const onChangeSelect = (event) => {
	filters.sortBy = event.target.value;
};
// поиск
const onChangeSearchInput = debounce((event) => {
	filters.searchQuery = event.target.value;
}, 500);
// добавление в закладки
const addToFavorite = async (item) => {
	try {
		// если нет в закладках
		if (!item.isFavorite) {
			const obj = {
				item_id: item.id,
				// item, //прокидываем объект для сохранения в закладку
			};
			item.isFavorite = true;
			const { data } = await axios.post(
				`https://280b3c98e5c903e5.mokky.dev/favorites`,
				obj
			);

			//сохраняем по id
			item.favoriteId = data.id;
		} else {
			item.isFavorite = false;
			// если есть в закладках
			await axios.delete(
				`https://280b3c98e5c903e5.mokky.dev/favorites/${item.favoriteId}`
			);

			item.favoriteId = null;
		}
	} catch (error) {
		console.log(error);
	}
};

//favorites наши закладки(избранное).
// получаем данные о закладках с бэка
const fetchFavorites = async () => {
	try {
		const { data: favorites } = await axios.get(
			`https://280b3c98e5c903e5.mokky.dev/favorites`
		);
		// ниже проверяем есть ли items в закладках
		items.value = items.value.map((item) => {
			const favorite = favorites.find(
				// favorite_id фишка mokky через связку 2х ресурсовв
				(favorite) => favorite.item_id === item.id
			);
			// если нет закладок - вернем item
			if (!favorite) {
				return item;
			}
			// если есть - вернем favorite с закладками
			return {
				...item,
				isFavorites: true,
				favoriteId: favorite.id,
			};
		});
	} catch (error) {
		console.log(error);
	}
};

// запрос на бэк, запрос списка товаров
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
		items.value = data.map((obj) => ({
			...obj,
			isFavorites: false,
			favoriteId: null,
			isAdded: false,
		}));
	} catch (error) {
		console.log(error);
	}
};

// функция для запроса с бэка
// при первом рендере приложения! с помощью хука обращаемся к бэку
onMounted(async () => {
	// получение списка кроссовок из localStorage
	const localCart = localStorage.getItem("cart");

	cart.value = localCart ? JSON.parse(localCart) : [];

	await fetchItems();
	await fetchFavorites(); // получение списка закладок
	// проверка есть ли данные в корзине
	items.value = items.value.map((item) => ({
		...item,
		isAdded: cart.value.some((cartItem) => cartItem.id === item.id),
	}));
});

// watch - следит за изменениями value в корзине
watch(cart, () => {
	items.value = items.value.map((item) => ({
		...item,
		isAdded: false,
	}));
});

// изменение сортировки по фильтру
// watch - следит за изменениями sortBy
watch(filters, fetchItems);
</script>

<template>
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
		<CardList
			:items="items"
			@add-to-favorite="addToFavorite"
			@add-to-cart="onClickAddPlus"
		/>
	</div>
</template>
