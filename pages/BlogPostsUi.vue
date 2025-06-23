<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRouter, useRuntimeConfig, useFetch, useToast } from '#imports'

interface Post {
  id: number
  title: string
  published_at: string
  user?: {
    name: string
  }
  category?: {
    title: string
  }
}

const config = useRuntimeConfig()
const apiBase = config.public.apiBase
const router = useRouter()
const toast = useToast()

const page = ref(1)
const pageSize = 10

const { data: postsData, pending: loading, error } = useFetch<Post[]>(`${apiBase}/blog/posts`)
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
  { key: 'published_at', label: 'Дата публікації' },
  { key: 'actions', label: 'Дії' }
]

const handleEdit = (id: number) => {
  router.push(`/blog/posts/${id}/edit`)
}

const handleDelete = async (id: number) => {
  if (confirm('Ви впевнені, що хочете видалити пост?')) {
    try {
      const res = await fetch(`${apiBase}/blog/posts/${id}`, {
        method: 'DELETE'
      })

      if (!res.ok) {
        throw new Error('Помилка видалення')
      }

      // Remove post from local state
      postsData.value = postsData.value?.filter(p => p.id !== id)

      toast.add({
        title: 'Успішно',
        description: 'Пост видалено',
        color: 'green'
      })
    } catch (err) {
      toast.add({
        title: 'Помилка',
        description: 'Не вдалося видалити пост',
        color: 'red'
      })
    }
  }
}
</script>

<template>
  <div class="relative w-full min-h-screen bg-gradient-to-br from-[#0f0c29] via-[#302b63] to-[#24243e] px-4 py-16">
    <div class="max-w-6xl mx-auto">
      <div class="flex justify-between mb-6">
        <h1 class="text-4xl font-bold text-white">Пости</h1>
        <UButton color="green" @click="router.push('/blog/posts/create')">
          Додати пост
        </UButton>
      </div>

      <UTable
          :rows="paginatedPosts"
          :columns="columns"
          :loading="loading"
          class="text-white"
      >
        <template #user.name-data="{ row }">
          <span>{{ row.user?.name ?? '—' }}</span>
        </template>

        <template #category.title-data="{ row }">
          <span>{{ row.category?.title ?? '—' }}</span>
        </template>

        <template #title-data="{ row }">
          <span class="font-medium">{{ row.title }}</span>
        </template>

        <template #published_at-data="{ row }">
          <span>{{ row.published_at ? new Date(row.published_at).toLocaleDateString('uk-UA') : '—' }}</span>
        </template>

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
            <UButton
                icon="i-heroicons-ellipsis-horizontal"
                color="gray"
                variant="ghost"
            />
          </UDropdown>
        </template>
      </UTable>

      <div class="flex justify-center pt-4">
        <UPagination
            v-model="page"
            :total="posts.length"
            :items-per-page="pageSize"
            class="text-white"
        />
      </div>
    </div>
  </div>
</template>