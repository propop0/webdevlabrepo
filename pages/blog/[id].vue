<script setup lang="ts">
import { useRoute } from 'vue-router'
import { useFetch } from '#app'

const route = useRoute()
const postId = route.params.id

interface Post {
  id: number
  title: string
  published_at: string
  content_raw: string
  user?: { name: string }
  category?: { title: string }
}

const config = useRuntimeConfig()
const apiBase = config.public.apiBase

const { data: post, pending, error } = await useFetch<Post>(`${apiBase}/blog/posts/${postId}`)
</script>

<template>
  <div class="min-h-screen px-6 py-12 bg-gradient-to-br from-[#0f0c29] via-[#302b63] to-[#24243e] text-white">
    <div class="max-w-4xl mx-auto backdrop-blur-md bg-white/10 p-6 rounded-xl border border-white/20 shadow-xl">
      <template v-if="post">
        <h1 class="text-3xl font-bold mb-2">{{ post.title }}</h1>
        <p class="text-sm text-gray-300 mb-4">
          {{ post.published_at }} | Автор: {{ post.user?.name ?? '—' }} | Категорія: {{ post.category?.title ?? '—' }}
        </p>
        <div class="text-lg whitespace-pre-line leading-relaxed">
          {{ post.content_raw }}
        </div>
      </template>

      <template v-else-if="pending">
        <p>Завантаження...</p>
      </template>

      <template v-else>
        <p>Пост не знайдено або сталася помилка.</p>
      </template>
    </div>
  </div>
</template>
