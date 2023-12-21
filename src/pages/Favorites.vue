<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";
import CardList from "../components/CardList.vue";

const favorites = ref([]);
// при первом рендере получаем данные о закладках с бэка
onMounted(async () => {
	try {
		const { data } = await axios.get(
			`https://280b3c98e5c903e5.mokky.dev/favorites?_relations=items`
		);
		favorites.value = data.map((obj) => obj.item);
	} catch (error) {
		console.log(error);
	}
});
</script>

<template>
	<h2 class="text-3xl font-bold mb-8">Мои закладки</h2>

	<CardList :items="favorites" is-favorites />
</template>
