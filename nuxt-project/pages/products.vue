<template>
  <div class="p-4">
    <h1 class="text-2xl font-bold mb-4">Список продуктів</h1>

    <UInput
        v-model="search"
        placeholder="Пошук продукту..."
        icon="i-heroicons-magnifying-glass-20-solid"
        class="mb-4"
    />

    <UTable
        :rows="sortedAndPaginated"
        :columns="columns"
        v-model:sort="sort"
        class="mb-4"
    />

    <UPagination
        v-model="page"
        :page-count="pageCount"
        :total="filtered.length"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

useHead({ title: 'Список продуктів' })

// Дані
const products = ref([])

// Колонки з сортуванням
const columns = [
  { id: 'id', key: 'id', label: 'ID', sortable: true },
  { id: 'name', key: 'name', label: 'Назва', sortable: true },
  { id: 'price', key: 'price', label: 'Ціна', sortable: true }
]


// Пошук
const search = ref('')
const filtered = computed(() =>
    products.value.filter(p =>
        p.name.toLowerCase().includes(search.value.toLowerCase())
    )
)

// Сортування
const sort = ref({ column: 'id', direction: 'asc' })
const sorted = computed(() => {
  const sorted = [...filtered.value]
  const { column, direction } = sort.value

  sorted.sort((a, b) => {
    if (a[column] < b[column]) return direction === 'asc' ? -1 : 1
    if (a[column] > b[column]) return direction === 'asc' ? 1 : -1
    return 0
  })

  return sorted
})

// Пагінація
const page = ref(1)
const pageSize = 5
const pageCount = computed(() => Math.ceil(filtered.value.length / pageSize))

const sortedAndPaginated = computed(() => {
  const start = (page.value - 1) * pageSize
  return sorted.value.slice(start, start + pageSize)
})
</script>
