<script setup lang="ts">
import { ref, computed } from 'vue'
import { useFetch, useRouter } from '#app'
import { useToast } from '#imports'
import { useRuntimeConfig } from '#app'

interface Category {
  id: number
  title: string
  parent_id?: number
  parent_category?: {
    id: number
    title: string
  }
}

const router = useRouter()
const toast = useToast()
const page = ref(1)
const pageSize = 10

const config = useRuntimeConfig()
const apiBase = config.public.apiBase

const { data: categoriesData, pending: loading, error } = await useFetch<Category[]>(`${apiBase}/blog/categories`)
const categories = computed(() => categoriesData.value ?? [])

const paginatedCategories = computed(() => {
  const start = (page.value - 1) * pageSize
  return categories.value.slice(start, start + pageSize)
})

const columns = [
  { key: 'id', label: '#' },
  { key: 'title', label: 'Назва категорії' },
  { key: 'parent_category.title', label: 'Батьківська категорія' },
  { key: 'actions', label: 'Дії' }
]

const handleEdit = (id: number) => {
  router.push(`/blog/categories/${id}/edit`)
}

const handleDelete = async (id: number) => {
  if (confirm('Ви впевнені, що хочете видалити категорію?')) {
    try {
      const res = await fetch(`${apiBase}/blog/categories/${id}`, {
        method: 'DELETE'
      })

      if (!res.ok) {
        throw new Error('Помилка видалення')
      }

      // забираємо локально з табл.
      categoriesData.value = categoriesData.value?.filter(cat => cat.id !== id)

      toast.add({
        title: 'Успіх',
        description: 'Категорію успішно видалено.',
        color: 'green'
      })
    } catch (err) {
      toast.add({
        title: 'Помилка',
        description: 'Не вдалося видалити категорію.',
        color: 'red'
      })
    }
  }
}
</script>

<template>
  <div class="relative w-full min-h-screen bg-gradient-to-br from-[#0f0c29] via-[#302b63] to-[#24243e] px-4 py-16">
    <div class="max-w-6xl mx-auto">
      <h1 class="text-4xl font-bold text-white text-center mb-10">Категорії блогу</h1>

      <div class="mb-4 text-right">
        <UButton color="green" @click="router.push('/blog/categories/create')">
          Додати категорію
        </UButton>
      </div>

      <UTable
          :rows="paginatedCategories"
          :columns="columns"
          :loading="loading"
          class="text-white"
      >
        <template #actions-data="{ row }">
          <UDropdown :items="[
            [
              {
                label: 'Редагувати',
                icon: 'i-heroicons-pencil',
                click: () => handleEdit(row.id)
              },
              {
                label: 'Видалити',
                icon: 'i-heroicons-trash',
                click: () => handleDelete(row.id)
              }
            ]
          ]">
            <UButton icon="i-heroicons-ellipsis-horizontal" color="gray" variant="ghost" />
          </UDropdown>
        </template>

        <template #parent_category.title-data="{ row }">
          <span>{{ row.parent_category?.title ?? '—' }}</span>
        </template>
      </UTable>

      <div class="flex justify-center pt-4">
        <UPagination
            v-model="page"
            :total="categories.length"
            :items-per-page="pageSize"
            class="text-white"
        />
      </div>
    </div>
  </div>
</template>