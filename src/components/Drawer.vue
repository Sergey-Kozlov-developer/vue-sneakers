<script setup>
import DrawerHead from "./DrawerHead.vue";
import CartItemList from "./CartItemList.vue";
import InfoBlock from "./InfoBlock.vue";

import axios from "axios";
import { ref, inject, computed } from "vue";

const props = defineProps({
	totalPrice: Number,
	vatPrice: Number,
	// buttonDisabled: Boolean,
});
// для закрытия drawer и получение корзины
const { cart } = inject("cart");
// оповещение о создании заказа
const isCreating = ref(false);
const orderId = ref(null);

// оформить заказ при клике на кнопку
const createOrder = async () => {
	try {
		isCreating.value = true;
		const { data } = await axios.post(
			"https://280b3c98e5c903e5.mokky.dev/orders",
			{
				items: cart.value,
				totalPrice: props.totalPrice.value,
			}
		);
		// очищаем корзину
		cart.value = [];

		orderId.value = data.id;
		return data;
	} catch (error) {
		console.log(error);
	} finally {
		isCreating.value = false;
	}
};
// проверка на пустую корзину
const cartIsEmpty = computed(() => cart.value.length === 0);

// проверка на создание заказа
const buttonDisabled = computed(() => isCreating.value || cartIsEmpty.value);
</script>
<template>
	<div
		class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"
	></div>
	<div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
		<DrawerHead />

		<div v-if="!totalPrice || orderId" class="flex h-full items-center">
			<InfoBlock
				v-if="!totalPrice && !orderId"
				title="Доставка"
				description="Добавьте хотя бы одну пару кроссовок, чтобы сделать заказ"
				image-url="/public/package-icon.png"
			/>
			<InfoBlock
				v-if="orderId"
				title="Заказ оформлен"
				:description="`Ваш заказ #${orderId} скоро будет передан курьерской доставке`"
				image-url="/public/order-success-icon.png"
			/>
		</div>

		<div v-else>
			<CartItemList />

			<div class="flex flex-col gap-4 mt-7">
				<div class="flex gap-2">
					<span>Итого:</span>
					<div class="flex-1 border-b border-dashed"></div>
					<b>{{ totalPrice }} руб.</b>
				</div>
				<div class="flex gap-2">
					<span>Налог 5%</span>
					<div class="flex-1 border-b border-dashed"></div>
					<b>{{ vatPrice }} руб.</b>
				</div>
				<button
					:disabled="buttonDisabled"
					@click="createOrder"
					class="mt-4 transition bg-lime-500 w-full rounded-xl py-3 text-white disabled:bg-slate-400 hover:bg-lime-600 active:bg-lime-700 cursor-pointer"
				>
					Оформить заказ
				</button>
			</div>
		</div>
	</div>
</template>
