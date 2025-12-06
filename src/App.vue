<script setup>
  import { ref, computed, onMounted, watch } from 'vue'
  const Products = [
    { id: 1, name: 'Ноутбук', category: 'Электроника', price: 50000, stock: 5 },
    { id: 2, name: 'Книга', category: 'Образование', price: 500, stock: 20 },
    { id: 3, name: 'Кофемашина', category: 'Бытовая техника', price: 15000, stock: 3 },
    { id: 4, name: 'Футболка', category: 'Одежда', price: 1000, stock: 15 },
    { id: 5, name: 'Наушники', category: 'Электроника', price: 3000, stock: 10 }
  ]
  const products = ref([])
  const searchQuery = ref('')
  const selectedCategory = ref('')
  const sortDirection = ref(null)

  const categories = computed(() => {
    return [... new Set(products.value.map(p => p.category))]
  })
  const filteredProducts = computed(() => {
    let result = products.value.filter(product => {
      const matchSearch = product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
      const matchCategory = selectedCategory.value ? product.category === selectedCategory.value : true
      return matchSearch && matchCategory
    })
    if (sortDirection.value === "asc") {
      result.sort((x, y) => x.price - y.price)
    } else if (sortDirection.value === "desc") {
      result.sort((x, y) => y.price - x.price)
    }
    return result
  })

  function buyProduct(id) {
    const product = products.value.find(p => p.id === id)
    if (product && product.stock > 0) {
      product.stock -= 1
      saveToLocalStorage()
    }
  }

  function sortAsc() {
    sortDirection.value = 'asc'
    saveFiltersToLocalStorage()
  }

  function sortDesc() {
    sortDirection.value = 'desc'
    saveFiltersToLocalStorage()
  }

  function resetFilters() {
    searchQuery.value = ''
    selectedCategory.value = ''
    sortDirection.value = null
    saveFiltersToLocalStorage()
  }

  const STORAGE_KEY_PRODUCTS = 'productCatalog_products'
  const STORAGE_KEY_FILTERS = 'productCatalog_filters'
  function loadFromLocalStorage() {
    const savedProducts = localStorage.getItem(STORAGE_KEY_PRODUCTS)
    if (savedProducts) {
      products.value = JSON.parse(savedProducts)
    } else {
      products.value = JSON.parse(JSON.stringify(Products))
    }

    const savedFilters = localStorage.getItem(STORAGE_KEY_FILTERS)
    if (savedFilters) {
      const { search, category, sort } = JSON.parse(savedFilters)
      searchQuery.value = search || ''
      selectedCategory.value = category || ''
      sortDirection.value = sort || null
    }
  }

  function saveToLocalStorage() {
    localStorage.setItem(STORAGE_KEY_PRODUCTS, JSON.stringify(products.value))
  }

  function saveFiltersToLocalStorage() {
    const filters = {
      search: searchQuery.value,
      category: selectedCategory.value,
      sort: sortDirection.value,
    }
    localStorage.setItem(STORAGE_KEY_FILTERS, JSON.stringify(filters))
  }

  watch(
    () => searchQuery.value,
    () => saveFiltersToLocalStorage(),
    { immediate: false }
  )

  watch(
    () => selectedCategory.value,
    () => saveFiltersToLocalStorage()
  )

  onMounted(() => {
    loadFromLocalStorage()
  })
</script>

<template>
  <div class="catalog">
    <input placeholder="Поиск по названию" type="text" v-model="searchQuery"/>
    <select v-model="selectedCategory">
      <option value="">Все категории</option>
      <option v-for="category in categories" :key="category" :value="category">
        {{ category }}
      </option>
    </select>
    <div class="sort-buttons">
      <button @click="sortAsc">Сортировка по возрастанию</button>
      <button @click="sortDesc">Сортировка по убыванию</button>
    </div>
    <button @click="resetFilters">Сбросить фильтры</button>
    <table class="table">
      <caption>
        Каталог товаров
      </caption>
      <thead>
        <tr>
          <th scope="col">Name</th>
          <th scope="col">Category</th>
          <th scope="col">Price</th>
          <th scope="col">Count</th>
          <th scope="col"></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in filteredProducts" :key="product.id">
          <th scope="row">{{ product.name }}</th>
          <td>{{ product.category }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.stock }}</td>
          <td>
             <button :disabled="product.stock <= 0" @click="buyProduct(product.id)" class="buy">Купить</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
  .table {
    width: 100%;
    border: none;
    margin-bottom: 20px;
  }
  .table thead th {
    padding: 10px;
    font-weight: 500;
    font-size: 16px;
    line-height: 20px;
    text-align: left;
    color: #444441;
    border-top: 2px solid #716561;
    border-bottom: 2px solid #716561;
  }
  .table tbody td {
    padding: 10px;
    font-size: 14px;
    line-height: 20px;
    color: white;
    border-top: 1px solid #716561;
  }
  .buy {
    border-radius: 10%;
    border: none;
    background-color: red;
    color: white;
  }
  .buy:hover {
    background-color: #eb4934;
    color: white;
  }
</style>
