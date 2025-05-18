<script setup>
import { onMounted, ref } from 'vue';
import axios from "axios";

import CardList from '../components/CardList.vue'

const favorites = ref([]);

onMounted(async () => {
  try {
    const { data } = await axios.get('https://6829871e6075e87073a6ba7a.mockapi.io/api/v1/favorites');

    favorites.value = data;
  } catch (err) {
    console.log(err);
  }
});
</script>

<template>
  <h2 class="text-3xl font-bold mb-8">Мої Закладки</h2>

  <CardList :items="favorites" is-favorites />
</template>