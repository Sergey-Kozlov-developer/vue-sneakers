<script setup>
import { ref, watch, provide, computed } from "vue";

import Header from "./components/Header.vue";

import Drawer from "./components/Drawer.vue";

/* КОРЗИНА */
// список товаров в корзине
const cart = ref([]);

// для открытия  drawer(корзины)
const drawerOpen = ref(false);
// общая стоимость
const totalPrice = computed(() =>
	cart.value.reduce((acc, item) => acc + item.price, 0)
);
// вычесление налога от стоимсоти заказа
const vatPrice = computed(() => Math.round((totalPrice.value * 5) / 100)); // 5%

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
	<Drawer v-if="drawerOpen" :total-price="totalPrice" :vat-price="vatPrice" />
	<div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
		<!-- <h1>Sergei</h1> -->
		<Header :total-price="totalPrice" @open-drawer="openDrawer" />
		<div class="p-10">
			<router-view></router-view>
		</div>
	</div>
</template>

<style scoped></style>
