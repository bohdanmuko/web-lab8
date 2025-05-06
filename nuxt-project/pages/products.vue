<script setup lang="ts">
import { h, resolveComponent, onMounted, computed, ref, watch } from 'vue'
import { upperFirst } from 'scule'
import type { TableColumn } from '@nuxt/ui'

useHead({
  title: 'Список продуктів'
})

// Розв'язання компонентів Nuxt UI
const UButton = resolveComponent('UButton')
const UCheckbox = resolveComponent('UCheckbox')
const UBadge = resolveComponent('UBadge')
const UDropdownMenu = resolveComponent('UDropdownMenu')
const ULoading = resolveComponent('ULoading') // Для завантажувального індикатора

const toast = useToast()

type Product = {
  id: number
  title: string
  description: string
  price: number
  discountPercentage: number
  rating: number
  stock: number
  brand: string
  category: string
  thumbnail: string
  images: string[]
}

const data = ref<Product[]>([])
const loading = ref(true)
const progress = ref(0)
// Використовуємо тип number для браузерного setInterval
const progressInterval = ref<number | null>(null)

const currentPage = ref(1)
const rowsPerPage = ref(10)

// Динамічний список варіантів для кількості рядків на сторінці
const rowsPerPageOptions = [5, 10, 20, 50]

// Стан сортування
const sorting = ref([
  {
    id: 'title',
    desc: false
  }
])

// Відсортовані дані (всі продукти)
const sortedData = computed(() => {
  // Якщо немає сортування або немає даних, повертаємо оригінальні дані
  if (!sorting.value.length || !data.value.length) {
    return data.value
  }

  // Клонуємо масив, щоб не змінювати оригінальні дані
  return [...data.value].sort((a, b) => {
    // Отримуємо поточне сортування
    const sort = sorting.value[0]!
    const columnId = sort.id
    const desc = sort.desc

    // Отримуємо значення для порівняння
    let valueA = a[columnId as keyof Product]
    let valueB = b[columnId as keyof Product]

    // Спеціальна обробка для числових значень
    if (typeof valueA === 'number' && typeof valueB === 'number') {
      return desc ? valueB - valueA : valueA - valueB
    }

    // Для рядків використовуємо localeCompare
    if (typeof valueA === 'string' && typeof valueB === 'string') {
      return desc ? valueB.localeCompare(valueA) : valueA.localeCompare(valueB)
    }

    // Для інших типів даних
    if (valueA === valueB) {
      return 0
    }

    // Якщо значення не рівні, порівнюємо їх
    if (valueA > valueB) {
      return desc ? -1 : 1
    }
    return desc ? 1 : -1
  })
})

// Обчислюваний список даних для поточної сторінки (тепер використовує відсортовані дані)
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * rowsPerPage.value
  const end = start + rowsPerPage.value
  return sortedData.value.slice(start, end)
})

// Загальна кількість сторінок
const totalPages = computed(() => Math.ceil(data.value.length / rowsPerPage.value))

// Скидаємо сторінку на першу при зміні сортування
watch(sorting, () => {
  currentPage.value = 1
})

// Функція для зміни сторінки
function changePage(page: number) {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page
  }
}

// Функція для зміни кількості рядків на сторінці
function updateRowsPerPage(newRowsPerPage: number | string) {
  rowsPerPage.value = Number(newRowsPerPage)
  currentPage.value = 1 // Повертаємося на першу сторінку
}

// Завантаження даних
async function fetchProducts() {
  loading.value = true
  progress.value = 0

  progressInterval.value = window.setInterval(() => {
    progress.value += Math.random() * 10
    if (progress.value >= 90) {
      progress.value = 90
      if (progressInterval.value !== null) {
        clearInterval(progressInterval.value)
      }
    }
  }, 200)

  try {
    const response = await fetch('https://dummyjson.com/products?limit=100')
    const result = await response.json()
    data.value = result.products

    progress.value = 100
    setTimeout(() => {
      loading.value = false
    }, 300)
  } catch (error) {
    console.error('Error fetching products:', error)
    toast.add({
      title: 'Помилка завантаження',
      color: 'error',
      icon: 'i-lucide-alert-circle'
    })
    loading.value = false
  } finally {
    if (progressInterval.value !== null) {
      clearInterval(progressInterval.value)
    }
  }
}

onMounted(() => {
  fetchProducts()
})


const columns: TableColumn<Product>[] = [
  {
    id: 'select',
    header: ({ table }) => h(UCheckbox, {
      'modelValue': table.getIsSomePageRowsSelected() ? 'indeterminate' : table.getIsAllPageRowsSelected(),
      'onUpdate:modelValue': (value: boolean | 'indeterminate') => table.toggleAllPageRowsSelected(!!value),
      'aria-label': 'Select all'
    }),
    cell: ({ row }) => h(UCheckbox, {
      'modelValue': row.getIsSelected(),
      'onUpdate:modelValue': (value: boolean | 'indeterminate') => row.toggleSelected(!!value),
      'aria-label': 'Select row'
    }),
    enableSorting: false,
    enableHiding: false
  },
  {
    accessorKey: 'title',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()

      return h(UButton, {
        color: 'neutral',
        variant: 'ghost',
        label: 'Назва',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        class: '-mx-2.5',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => h('div', { class: 'font-medium' }, row.getValue('title'))
  },
  {
    accessorKey: 'description',
    header: 'Опис',
    cell: ({ row }) => h('div', { class: 'max-w-md truncate' }, row.getValue('description'))
  },
  {
    accessorKey: 'price',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()

      return h(UButton, {
        color: 'neutral',
        variant: 'ghost',
        label: 'Ціна',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        class: '-mx-2.5',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => {
      const price = Number.parseFloat(row.getValue('price'))
      const formatted = new Intl.NumberFormat('uk-UA', {
        style: 'currency',
        currency: 'USD'
      }).format(price)
      return h('div', { class: 'font-medium' }, formatted)
    }
  },
  {
    accessorKey: 'rating',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()

      return h(UButton, {
        color: 'neutral',
        variant: 'ghost',
        label: 'Оцінка',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        class: '-mx-2.5',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => {
      const rating = Number(row.getValue('rating'))
      const textColor = rating >= 4.5 ? 'text-green-500' : 'text-red-500'
      return h('div', { class: `font-medium ${textColor}` }, rating.toFixed(1))
    }
  },
  {
    accessorKey: 'brand',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()

      return h(UButton, {
        color: 'neutral',
        variant: 'ghost',
        label: 'Бренд',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        class: '-mx-2.5',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => h('div', {}, row.getValue('brand'))
  },
  {
    accessorKey: 'category',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()

      return h(UButton, {
        color: 'neutral',
        variant: 'ghost',
        label: 'Категорія',
        icon: isSorted
            ? isSorted === 'asc'
                ? 'i-lucide-arrow-up-narrow-wide'
                : 'i-lucide-arrow-down-wide-narrow'
            : 'i-lucide-arrow-up-down',
        class: '-mx-2.5',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => h(UBadge, { class: 'capitalize', variant: 'subtle', color: 'info' }, () => row.getValue('category'))
  },
  {
    accessorKey: 'thumbnail', // Фотографії
    header: 'Фото',
    cell: ({ row }) => h('img', {
      src: row.getValue('thumbnail'),
      alt: row.getValue('title'),
      class: 'w-24 h-24 object-cover rounded',
      style: 'width: 100px; height: 100px;'
    }),
    enableSorting: false
  }
]

const table = useTemplateRef('table')

function randomize() {
  data.value = [...data.value].sort(() => Math.random() - 0.5)
}

function reloadData() {
  fetchProducts()
}

const skeletonRows = Array(10).fill(0).map((_, i) => i)
</script>

<template>
  <div class="flex-1 divide-y divide-(--ui-border-accented) w-full h-screen">
    <!-- Лінія прогресу при завантаженні -->
    <div v-if="loading" class="w-full h-1 bg-gray-200 dark:bg-gray-700 fixed top-0 left-0 z-50">
      <div class="h-1 bg-blue-500 transition-all duration-300 ease-out" :style="{ width: `${progress}%` }"></div>
    </div>

    <div class="flex items-center gap-2 px-4 py-3.5 overflow-x-auto">
      <UInput
          :model-value="(table?.tableApi?.getColumn('title')?.getFilterValue() as string)"
          class="max-w-sm min-w-[12ch]"
          placeholder="Пошук за назвою..."
          @update:model-value="table?.tableApi?.getColumn('title')?.setFilterValue($event)"
      />
      <UButton color="neutral" label="Випадково" @click="randomize" />
      <UButton color="info" label="Оновити" icon="i-lucide-refresh-cw" @click="reloadData" />

      <select
          v-model="rowsPerPage"
          @change="updateRowsPerPage(($event.target as HTMLSelectElement).value)"
          class="border rounded px-2 py-1"
      >
        <option v-for="option in rowsPerPageOptions" :key="option" :value="option">
          {{ option }} рядків
        </option>
      </select>

      <UDropdownMenu
          :items="table?.tableApi?.getAllColumns().filter(column => column.getCanHide()).map(column => ({
          label: upperFirst(column.id),
          type: 'checkbox' as const,
          checked: column.getIsVisible(),
          onUpdateChecked(checked: boolean) {
            table?.tableApi?.getColumn(column.id)?.toggleVisibility(!!checked)
          },
          onSelect(e?: Event) {
            e?.preventDefault()
          }
        }))"
          :content="{ align: 'end' }"
      >
        <UButton
            label="Колонки"
            color="neutral"
            variant="outline"
            trailing-icon="i-lucide-chevron-down"
            class="ml-auto"
            aria-label="Columns select dropdown"
        />
      </UDropdownMenu>
    </div>

    <!-- Секція таблиці -->
    <div v-if="loading" class="relative w-full h-full">
      <!-- Loading overlay -->
      <div class="absolute inset-0 flex flex-col items-center justify-center bg-white dark:bg-gray-800 bg-opacity-80 dark:bg-opacity-80 z-10">
        <ULoading size="lg" class="mb-4" />
        <p class="text-sm text-gray-600 dark:text-gray-300">Завантаження даних про продукти...</p>
        <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">{{ Math.round(progress) }}% завершено</p>
      </div>
      <!-- Skeleton таблиця -->
      <table class="w-full table-auto">
        <thead>
        <tr class="border-b border-gray-200 dark:border-gray-700">
          <th class="p-3 text-left">
            <div class="h-5 w-5 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-32 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-16 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-16 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-20 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="index in skeletonRows" :key="index" class="border-b border-gray-200 dark:border-gray-700">
          <td class="p-3">
            <div class="h-5 w-5 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-24 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-32 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-48 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-16 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-10 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
        </tr>
        </tbody>
      </table>
    </div>

    <!-- UTable займає всю доступну висоту -->
    <UTable
        v-else
        ref="table"
        :data="paginatedData"
        :columns="columns"
        v-model:sorting="sorting"
        sticky
        class="h-full"
    >
      <template #expanded="{ row }">
        <div class="p-4 bg-gray-50 dark:bg-gray-800">
          <div class="flex gap-4">
            <div class="flex space-x-2">
              <img v-for="(image, idx) in row.original.images.slice(0, 3)" :key="idx"
                   :src="image" :alt="`${row.original.title} - image ${idx + 1}`"
                   class="w-32 h-32 object-cover rounded"
              />
            </div>
            <div>
              <h3 class="text-lg font-semibold">{{ row.original.title }}</h3>
              <p class="text-sm text-gray-600 dark:text-gray-300 mt-1">{{ row.original.description }}</p>
              <div class="mt-2 flex flex-wrap gap-2">
                <div>
                  <span class="text-sm font-medium">Знижка: </span>
                  <UBadge color="success">{{ row.original.discountPercentage }}% off</UBadge>
                </div>
                <div>
                  <span class="text-sm font-medium">Залишок: </span>
                  <UBadge :color="row.original.stock > 50 ? 'success' : 'warning'">{{ row.original.stock }} шт.</UBadge>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>
    </UTable>

    <!-- Елементи пагінації -->
    <div class="flex justify-between items-center px-4 py-3">
      <span class="text-sm text-gray-600 dark:text-gray-300">
        Сторінка {{ currentPage }} з {{ totalPages }}
      </span>
      <div class="flex items-center gap-2">
        <UButton
            :disabled="currentPage === 1"
            label="Попередня"
            @click="changePage(currentPage - 1)"
        />
        <UButton
            :disabled="currentPage === totalPages"
            label="Наступна"
            @click="changePage(currentPage + 1)"
        />
      </div>
    </div>

    <div class="px-4 py-3.5 text-sm text-(--ui-text-muted)">
      {{ table?.tableApi?.getFilteredSelectedRowModel().rows.length || 0 }} з
      {{ table?.tableApi?.getFilteredRowModel().rows.length || 0 }} рядків вибрано.
    </div>
  </div>
</template>