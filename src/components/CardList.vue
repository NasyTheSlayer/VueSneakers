<script setup>
import Card from './Card.vue';

defineProps({
  items: Array,
  isFavorites: Boolean,
})

const emit = defineEmits(['addToFavorite', 'addToCart']);
</script>

<template>
  <div class="grid grid-cols-4 gap-5" v-auto-animate>
    <Card :key="item.id" v-for="item in items" :image-url="item.imageUrl" :title="item.title" :price="item.price"
      :isFavorite="item.isFavorite" :id="item.id"
      :onClickFavorite="isFavorites ? null : () => emit('addToFavorite', item)"
      :onClickAdd="isFavorites ? null : () => emit('addToCart', item)" :is-added="item.isAdded" />
  </div>
</template>