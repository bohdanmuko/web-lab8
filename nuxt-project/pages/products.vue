<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

type Product = {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const data = ref<Product[]>([])
const searchQuery = ref('')
const currentPage = ref(1)
const itemsPerPage = 5
const sortColumn = ref('title')
const sortDirection = ref<'asc' | 'desc'>('asc')

// Fetch data from API
onMounted(async () => {
  try {
    const res = await fetch('https://dummyjson.com/products')
    const json = await res.json()
    data.value = json.products
  } catch (error) {
    console.error('Error fetching products:', error)
  }
})

// Filter data based on search query
const filteredData = computed(() => {
  if (!searchQuery.value) return data.value

  const query = searchQuery.value.toLowerCase()
  return data.value.filter(product =>
      product.title.toLowerCase().includes(query)
  )
})

// Sort data based on column and direction
const sortedData = computed(() => {
  return [...filteredData.value].sort((a, b) => {
    const aValue = a[sortColumn.value as keyof Product]
    const bValue = b[sortColumn.value as keyof Product]

    if (typeof aValue === 'string' && typeof bValue === 'string') {
      return sortDirection.value === 'asc'
          ? aValue.localeCompare(bValue)
          : bValue.localeCompare(aValue)
    } else {
      return sortDirection.value === 'asc'
          ? Number(aValue) - Number(bValue)
          : Number(bValue) - Number(aValue)
    }
  })
})

// Paginate data
const paginatedData = computed(() => {
  const startIndex = (currentPage.value - 1) * itemsPerPage
  return sortedData.value.slice(startIndex, startIndex + itemsPerPage)
})

// Total pages
const totalPages = computed(() =>
    Math.ceil(filteredData.value.length / itemsPerPage)
)

// Handle sorting
const handleSort = (column: string) => {
  if (sortColumn.value === column) {
    sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortColumn.value = column
    sortDirection.value = 'asc'
  }
}

// Get rating color
const getRatingColor = (rating: number) => {
  return rating < 4.5 ? 'red' : 'green'
}
</script>

<template>
  <div class="product-table-container">
    <div class="search-container">
      <input
          v-model="searchQuery"
          class="search-input"
          placeholder="Пошук по назві..."
      />
    </div>

    <table class="product-table">
      <thead>
      <tr>
        <th @click="handleSort('title')" class="sortable-header">
          Назва
          <span class="sort-icon" v-if="sortColumn === 'title'">
              {{ sortDirection === 'asc' ? '▲' : '▼' }}
            </span>
        </th>
        <th @click="handleSort('description')" class="sortable-header">
          Опис
          <span class="sort-icon" v-if="sortColumn === 'description'">
              {{ sortDirection === 'asc' ? '▲' : '▼' }}
            </span>
        </th>
        <th @click="handleSort('price')" class="sortable-header">
          Ціна
          <span class="sort-icon" v-if="sortColumn === 'price'">
              {{ sortDirection === 'asc' ? '▲' : '▼' }}
            </span>
        </th>
        <th @click="handleSort('rating')" class="sortable-header">
          Оцінка
          <span class="sort-icon" v-if="sortColumn === 'rating'">
              {{ sortDirection === 'asc' ? '▲' : '▼' }}
            </span>
        </th>
        <th @click="handleSort('brand')" class="sortable-header">
          Бренд
          <span class="sort-icon" v-if="sortColumn === 'brand'">
              {{ sortDirection === 'asc' ? '▲' : '▼' }}
            </span>
        </th>
        <th @click="handleSort('category')" class="sortable-header">
          Категорія
          <span class="sort-icon" v-if="sortColumn === 'category'">
              {{ sortDirection === 'asc' ? '▲' : '▼' }}
            </span>
        </th>
        <th>Фото</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="product in paginatedData" :key="product.id">
        <td>{{ product.title }}</td>
        <td class="description-cell">{{ product.description }}</td>
        <td>₴{{ product.price }}</td>
        <td>
            <span class="badge" :class="getRatingColor(product.rating)">
              {{ product.rating }}
            </span>
        </td>
        <td>{{ product.brand }}</td>
        <td>{{ product.category }}</td>
        <td>
          <img
              :src="product.thumbnail"
              width="100"
              height="100"
              alt="Фото продукту"
              class="product-image"
          />
        </td>
      </tr>
      <tr v-if="paginatedData.length === 0">
        <td colspan="7" class="empty-message">Немає даних для відображення</td>
      </tr>
      </tbody>
    </table>

    <div class="pagination" v-if="totalPages > 1">
      <button
          class="pagination-button"
          :disabled="currentPage === 1"
          @click="currentPage = Math.max(1, currentPage - 1)"
      >
        Попередня
      </button>
      <span class="page-info">{{ currentPage }} / {{ totalPages }}</span>
      <button
          class="pagination-button"
          :disabled="currentPage === totalPages"
          @click="currentPage = Math.min(totalPages, currentPage + 1)"
      >
        Наступна
      </button>
    </div>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');

.product-table-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 1000px; /* Add this line to make the table narrower */
  margin: 0 auto; /* Center the table */
  flex: 1;
  font-family: 'Poppins', sans-serif;
  background-color: #f8f9ff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

.search-container {
  display: flex;
  padding: 20px;
  background: linear-gradient(135deg, #1e338d, #180037);
}

.search-input {
  max-width: 500px;
  width: 100%;
  padding: 12px 16px;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-family: 'Poppins', sans-serif;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  outline: none;
  margin: 0 auto;
}

.search-input:focus {
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
}

.product-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  table-layout: fixed;
}

.product-table th,
.product-table td {
  padding: 12px; /* Reduced from 16px */
  text-align: left;
}

.product-table th {
  font-weight: 600;
  background-color: #f0f2ff;
  color: #4a5568;
  position: sticky;
  top: 0;
  z-index: 10;
  border-bottom: 2px solid #e2e8f0;
}

/* Add this new CSS rule for column widths */
.product-table th:nth-child(1) { width: 15%; } /* Title */
.product-table th:nth-child(2) { width: 20%; } /* Description */
.product-table th:nth-child(3) { width: 8%; }  /* Price */
.product-table th:nth-child(4) { width: 8%; }  /* Rating */
.product-table th:nth-child(5) { width: 12%; } /* Brand */
.product-table th:nth-child(6) { width: 12%; } /* Category */
.product-table th:nth-child(7) { width: 15%; } /* Photo */

.product-table tr:nth-child(even) {
  background-color: #f8faff;
}

.product-table tr:hover {
  background-color: #eef2ff;
}

.sortable-header {
  cursor: pointer;
  user-select: none;
  position: relative;
  transition: all 0.2s;
}

.sortable-header:hover {
  background-color: #e4e9ff;
  color: #5a67d8;
}

.sort-icon {
  margin-left: 4px;
  font-size: 12px;
  color: #6366f1;
}

.description-cell {
  max-width: 200px; /* Reduced from 300px */
  white-space: normal; /* Change this line to allow text to wrap */
  overflow: hidden;
  text-overflow: ellipsis;
  color: #4b5563;
}


.badge {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 9999px;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 0.5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.badge.red {
  background: linear-gradient(135deg, #ff9a9e, #fad0c4);
  color: #c53030;
}

.badge.green {
  background: linear-gradient(135deg, #84fab0, #8fd3f4);
  color: #276749;
}

.product-image {
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
  width: 80px; /* Add this line */
  height: 80px; /* Add this line */
}

.product-image:hover {
  transform: scale(1.05);
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  gap: 16px;
  background: linear-gradient(135deg, #1e338d, #180037);
}

.pagination-button {
  padding: 8px 16px;
  border: none;
  background-color: white;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-family: 'Poppins', sans-serif;
  font-weight: 500;
  color: #4c51bf;
  transition: all 0.2s;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.pagination-button:hover:not(:disabled) {
  background-color: #f0f4ff;
  transform: translateY(-2px);
  box-shadow: 0 6px 10px rgba(0, 0, 0, 0.15);
}

.pagination-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.page-info {
  font-size: 14px;
  font-weight: 600;
  color: white;
  background-color: rgba(255, 255, 255, 0.2);
  padding: 6px 12px;
  border-radius: 8px;
}

.empty-message {
  text-align: center;
  padding: 40px;
  color: #6b7280;
  font-style: italic;
}
</style>