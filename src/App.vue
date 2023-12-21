<script setup>
import { ref, watch, provide, computed } from "vue";
import axios from "axios";

import Header from "./components/Header.vue";

import Drawer from "./components/Drawer.vue";

/* КОРЗИНА */
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
// для закрытия drawer
const closeDrawer = () => {
	drawerOpen.value = false;
};
// для открытия drawer
const openDrawer = () => {
	drawerOpen.value = true;
};

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

/* КОРЗИНА end */
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
			<router-view></router-view>
		</div>
	</div>
</template>

<style scoped></style>
