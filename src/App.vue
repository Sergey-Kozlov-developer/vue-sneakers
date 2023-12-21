<script setup>
import { onMounted, ref, watch, reactive, provide, computed } from "vue";
import axios from "axios";

import Header from "./components/Header.vue";
import CardList from "./components/CardList.vue";
import Drawer from "./components/Drawer.vue";

const items = ref([]);
// список товаров в корзине
const cart = ref([]);
// оповещение о создании заказа
const isCreatingOrder = ref(false);

// для открытия  drawer(корзины)
const drawerOpen = ref(false);
// общая стоимость
const totalPrice = computed(() =>
	cart.value.reduce((acc, item) => acc + item.price, 0)
);
// вычесление налога от стоимсоти заказа
const vatPrice = computed(() => Math.round((totalPrice.value * 5) / 100)); // 5%
// проверка на пустую корзину
const cartIsEmpty = computed(() => cart.value.length === 0);
// проверка на создание заказа
const cartButtonDisabled = computed(
	() => isCreatingOrder.value || cartIsEmpty.value
);
const closeDrawer = () => {
	drawerOpen.value = false;
};
const openDrawer = () => {
	drawerOpen.value = true;
};
// для фильтрации и поиска
const filters = reactive({
	sortBy: "title",
	searchQuery: "",
});
// для добавления в корзину
const addToCart = (item) => {
	cart.value.push(item);
	item.isAdded = true;
};
const removeFromCart = (item) => {
	cart.value.splice(cart.value.indexOf(item), 1);
	item.isAdded = false;
};
// оформить заказ при клике на кнопку
const createOrder = async () => {
	try {
		isCreatingOrder.value = true;
		const { data } = await axios.post(
			"https://280b3c98e5c903e5.mokky.dev/orders",
			{
				items: cart.value,
				totalPrice: totalPrice.value,
			}
		);
		// очищаем корзину
		cart.value = [];
		return data;
	} catch (error) {
		console.log(error);
	} finally {
		isCreatingOrder.value = false;
	}
};
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
const onChangeSearchInput = (event) => {
	filters.searchQuery = event.target.value;
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
				(favorite) => favorite.parentId === item.id
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

// добавление в закладки
const addToFavorite = async (item) => {
	try {
		// если нет в закладках
		if (!item.isFavorite) {
			const obj = {
				parentId: item.id,
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
// изменение сортировки по фильтру
// watch - следит за изменениями sortBy
watch(filters, fetchItems);
// watch - следит за изменениями value в корзине
watch(cart, () => {
	items.value = items.value.map((item) => ({
		...item,
		isAdded: false,
	}));
});
// watch - следит за изменениями в корзине и схораняет в localStorage(на клиенте в браузере)
watch(
	cart,
	() => {
		localStorage.setItem("cart", JSON.stringify(cart.value));
	},
	{ deep: true }
);

// прокидывание данных между компонентами без props
// для открытия и закрытия окна корзины
provide("cart", {
	cart,
	closeDrawer,
	openDrawer,
	addToCart,
	removeFromCart,
});
</script>

<template>
	<Drawer
		v-if="drawerOpen"
		:total-price="totalPrice"
		:vat-price="vatPrice"
		@create-order="createOrder"
		@close-drawer="closeDrawer"
		:button-disabled="cartButtonDisabled"
	/>
	<div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
		<!-- <h1>Sergei</h1> -->
		<Header :total-price="totalPrice" @open-drawer="openDrawer" />
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
				<CardList
					:items="items"
					@add-to-favorite="addToFavorite"
					@add-to-cart="onClickAddPlus"
				/>
			</div>
		</div>
	</div>
</template>

<style scoped></style>
