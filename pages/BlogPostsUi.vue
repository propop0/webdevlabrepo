<template>
  <div class="relative w-full min-h-screen bg-gradient-to-br from-[#0f0c29] via-[#302b63] to-[#24243e] px-4 py-16">
    <div class="max-w-6xl mx-auto flex items-center justify-center h-[120px] mb-10">
      <h1
          class="text-4xl font-bold text-white text-center tracking-tight px-6 py-4 rounded-xl border border-white/20 shadow-lg backdrop-blur-md bg-white/10 flex items-center justify-center"
      >
        Пости блогу
      </h1>
    </div>

    <div class="max-w-6xl mx-auto space-y-6 backdrop-blur-sm bg-white/5 border border-white/10 p-6 rounded-xl shadow-2xl">
      <div class="overflow-x-auto rounded-lg border border-white/10 backdrop-blur-md bg-white/5 shadow-xl">
        <UTable
            :rows="paginatedPosts"
            :columns="columns"
            :loading="loading"
            class="min-w-full text-white"
        >
          <template #user.name-data="{ row }">
            <span class="text-white">
              {{ row.user?.name ?? '—' }}
            </span>
          </template>
          <template #category.title-data="{ row }">
            <span class="text-white">
              {{ row.category?.title ?? '—' }}
            </span>
          </template>
          <template #title-data="{ row }">
            <NuxtLink :to="`/blog/${row.id}`" class="text-blue-400 hover:underline">
              {{ row.title }}
            </NuxtLink>
          </template>
        </UTable>
      </div>

      <div class="flex justify-center pt-4 border-t border-white/10">
        <UPagination
            v-model="page"
            :total="posts.length"
            :items-per-page="pageSize"
            class="bg-white/10 backdrop-blur-md text-white px-4 py-2 rounded-lg shadow-lg"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useFetch, useRuntimeConfig } from '#app'

interface Post {
  id: number
  title: string
  published_at: string
  user?: { name: string }
  category?: { title: string }
}

const config = useRuntimeConfig()
const apiBase = config.public.apiBase

const page = ref(1)
const pageSize = 10

const { data: postsData, pending: loading, error } = await useFetch<Post[]>(`${apiBase}/blog/posts`)

const posts = computed(() => postsData.value ?? [])

const paginatedPosts = computed(() => {
  const start = (page.value - 1) * pageSize
  return posts.value.slice(start, start + pageSize)
})

const columns = [
  { key: 'id', label: '#' },
  { key: 'user.name', label: 'Автор' },
  { key: 'category.title', label: 'Категорія' },
  { key: 'title', label: 'Заголовок' },
  { key: 'published_at', label: 'Дата публікації' }
]
</script>

