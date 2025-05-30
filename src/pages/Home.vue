<script setup>
import axios from 'axios';
import debounce from 'lodash.debounce';
import { inject, reactive, watch, ref, onMounted } from 'vue';
import CardList from '../components/CardList.vue';

const { cart, addToCart, removeFromCart } = inject('cart')

const items = ref([]);

const filters = reactive({
  sortBy: 'title',
  searchQuery: '',
})


const onChangeSelect = (event) => {
  filters.sortBy = event.target.value;
}

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value;
}, 300)

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item);
  }
}

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id,
        title: item.title,
        price: item.price,
        imageUrl: item.imageUrl
      };
      item.isFavorite = true;

      const { data } = await axios.post(`https://6829871e6075e87073a6ba7a.mockapi.io/api/v1/favorites`, obj)

      item.favoriteId = data.id;
    } else {
      item.isFavorite = false;
      await axios.delete(`https://6829871e6075e87073a6ba7a.mockapi.io/api/v1/favorites/${item.favoriteId}`)
      item.favoriteId = null;
    }
  } catch (err) {
    console.log(err);
  }
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://6829871e6075e87073a6ba7a.mockapi.io/api/v1/favorites`)

    items.value = items.value.map(item => {
      const favorite = favorites.find(favorite => favorite.item_id === item.id);

      if (!favorite) {
        return item;
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id,
      }
    });
  } catch (err) {
    console.log(err);
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      params.title = `${filters.searchQuery}`;
    }

    const { data } = await axios.get(`https://6829871e6075e87073a6ba7a.mockapi.io/api/v1/items`, {
      params
    })

    items.value = data.map(obj => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false,
    }));
  } catch (err) {
    console.log(err);
  }
}

onMounted(async () => {
  const localCart = localStorage.getItem('cart');
  cart.value = localCart ? JSON.parse(localCart) : [];

  await fetchItems();
  await fetchFavorites();

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
});

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false,
  }))
});

watch(filters, fetchItems);
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

    <div class="flex gap-4">
      <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
        <option value="name">По назві</option>
        <option value="price">По ціні(дорогі)</option>
        <option value="-price">По ціні(дешеві)</option>
      </select>

      <div class="relative">
        <img class="absolute left-4 top-3" src="/search.svg" alt="Search">
        <input @input="onChangeSearchInput" class="border rounded-md py-1.5 pl-11 pr-4 outline-none focus:border-gray-400"
          type="text" placeholder="Пошук...">
      </div>
    </div>
  </div>

  <div class="mt-10">
    <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
  </div>
</template>