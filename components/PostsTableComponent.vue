<template>
  <div class="container">
    <div class="flex justify-center">
      <div class="w-full">
        <nav class="navbar bg-gray-100 p-4">
          <a href="/admin/blog/posts/create" class="text-blue-500 underline">Додати</a>
        </nav>

        <div class="card">
          <div class="card-body">
            <table class="table-auto w-full border">
              <thead>
              <tr class="bg-gray-200">
                <th class="p-2 border">#</th>
                <th class="p-2 border">Автор</th>
                <th class="p-2 border">Категорія</th>
                <th class="p-2 border">Заголовок</th>
                <th class="p-2 border">Дата публікації</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="post in posts" :key="post.id">
                <td class="p-2 border">{{ post.id }}</td>
                <td class="p-2 border">{{ post.user?.name }}</td>
                <td class="p-2 border">{{ post.category?.title }}</td>
                <td class="p-2 border">
                  <a :href="'/admin/blog/posts/' + post.id + '/edit'" class="text-blue-600 hover:underline">
                    {{ post.title }}
                  </a>
                </td>
                <td class="p-2 border">{{ post.published_at }}</td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

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

const posts = ref<Post[]>([])
const config = useRuntimeConfig()
const apiBase = config.public.apiBase

const getPosts = async () => {
  try {
    const response = await $fetch<Post[]>(`${apiBase}/blog/posts`)
    posts.value = response
  } catch (e) {
    console.error('Помилка завантаження постів', e)
  }
}

onMounted(() => {
  getPosts()
})
</script>
