<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useFetch } from '#app'
import type { DataTableColumns } from '#ui/types'

interface Product {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const search = ref('')
const page = ref(1)
const pageSize = 10

const sort = ref({
  kriteriy: 'title',
  direction: 'asc'
})

const zminaNapryamku = () => {
  sort.value.direction = sort.value.direction === 'asc' ? 'desc' : 'asc'
}

const { data, pending, error } = await useFetch<{ products: Product[] }>('https://dummyjson.com/products?limit=100')

const allProducts = computed(() => data.value?.products || [])
const loading = pending

const vidibraniTovary = computed(() => {
  if (!search.value) return allProducts.value
  return allProducts.value.filter(product =>
      Object.values(product).some(val =>
          String(val).toLowerCase().includes(search.value.toLowerCase())
      )
  )
})

const sortovaniTovary = computed(() => {
  const items = [...vidibraniTovary.value]
  const { kriteriy, direction } = sort.value
  return items.sort((a, b) => {
    const aVal = a[kriteriy as keyof Product]
    const bVal = b[kriteriy as keyof Product]

    if (typeof aVal === 'string' && typeof bVal === 'string') {
      return direction === 'asc' ? aVal.localeCompare(bVal) : bVal.localeCompare(aVal)
    }
    if (typeof aVal === 'number' && typeof bVal === 'number') {
      return direction === 'asc' ? aVal - bVal : bVal - aVal
    }
    return 0
  })
})

const paginatedProducts = computed(() => {
  const start = (page.value - 1) * pageSize
  return sortovaniTovary.value.slice(start, start + pageSize)
})

watch([vidibraniTovary, sort], () => {
  page.value = 1
})

const columns: DataTableColumns<Product> = [
  { key: 'title', label: 'Назва' },
  {
    key: 'description',
    label: 'Опис',
    class: 'max-w-[400px] whitespace-normal text-sm text-gray-300'
  },
  { key: 'price', label: 'Ціна' },
  {
    key: 'rating',
    label: 'Оцінка',
    class: 'text-sm',
    sortable: true
  },
  { key: 'brand', label: 'Бренд' },
  { key: 'category', label: 'Категорія' },
  {
    key: 'thumbnail',
    label: 'Фото'
  }
]
</script>

<template>
  <div class="space-y-6 backdrop-blur-sm bg-white/5 border border-white/10 p-6 rounded-xl shadow-2xl">
    <!-- Фільтр і сортування -->
    <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
      <UInput
          v-model="search"
          placeholder="🔍 Пошук продуктів..."
          class="w-full md:max-w-sm backdrop-blur-md bg-white/10 border border-white/20 rounded-xl shadow-lg text-white placeholder-gray-300"
      />

      <div class="flex gap-3 items-center">
        <label class="text-white text-sm font-medium">Сортувати за:</label>
        <div class="relative">
          <select
              v-model="sort.kriteriy"
              class="bg-white/10 text-white border border-white/20 px-4 py-2 rounded-lg backdrop-blur-md appearance-none pr-10"
          >
            <option class="bg-gray-800 text-white" value="title">Назва</option>
            <option class="bg-gray-800 text-white" value="price">Ціна</option>
            <option class="bg-gray-800 text-white" value="rating">Оцінка</option>
            <option class="bg-gray-800 text-white" value="brand">Бренд</option>
            <option class="bg-gray-800 text-white" value="category">Категорія</option>
          </select>

          <div class="pointer-events-none absolute inset-y-0 right-3 flex items-center text-white">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M19 9l-7 7-7-7" />
            </svg>
          </div>
        </div>

        <button
            @click="zminaNapryamku"
            class="px-4 py-2 rounded-lg bg-gradient-to-r from-indigo-500 to-purple-600 hover:from-purple-700 hover:to-indigo-700 text-white font-medium shadow-md transition-all duration-200"
        >
          {{ sort.direction === 'asc' ? 'за спаданням ⇩' : 'за зростанням ⇧' }}
        </button>
      </div>
    </div>

    <div class="overflow-x-auto rounded-lg border border-white/10 backdrop-blur-md bg-white/5 shadow-xl">
      <UTable
          :rows="paginatedProducts"
          :columns="columns"
          :loading="loading"
          sort-mode="manual"
          class="min-w-full text-white"
      >
        <template #description-data="{ row }">
          <div class="whitespace-normal text-sm max-w-[400px] text-gray-200">
            {{ row.description }}
          </div>
        </template>

        <template #rating-data="{ row }">
          <span :class="row.rating < 4.5 ? 'text-red-400' : 'text-green-400 font-bold'">
            {{ row.rating.toFixed(1) }}
          </span>
        </template>

        <template #thumbnail-data="{ row }">
          <div class="flex justify-center">
            <img
                :src="row.thumbnail"
                :alt="row.title"
                class="w-[100px] h-[100px] object-cover rounded-xl shadow-lg border border-white/20 backdrop-blur-md"
            />
          </div>
        </template>
      </UTable>
    </div>

    <div class="flex justify-center pt-4 border-t border-white/10">
      <UPagination
          v-model="page"
          :total="sortovaniTovary.length"
          :items-per-page="pageSize"
          class="bg-white/10 backdrop-blur-md text-white px-4 py-2 rounded-lg shadow-lg"
      />
    </div>
  </div>
</template>
