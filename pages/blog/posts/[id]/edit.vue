<script setup lang="ts">
import { ref, watch } from 'vue'
import { useRoute, useRouter, useRuntimeConfig } from '#app'
import { useToast, useFetch } from '#imports'
import { z } from 'zod'
import { $fetch } from 'ofetch'

interface Category {
  id: number
  title: string
}

interface BlogPost {
  id: number
  title: string
  content_raw: string
  category_id: number
  published_at: string | null
}

const route = useRoute()
const router = useRouter()
const config = useRuntimeConfig()
const toast = useToast()

const postId = Number(route.params.id)

const schema = z.object({
  title: z.string().min(3, 'Заголовок має містити щонайменше 3 символи'),
  content: z.string().min(10, 'Контент має містити щонайменше 10 символів'),
  category_id: z.coerce.number({ required_error: 'Оберіть категорію' }),
  published_at: z.string().optional()
})

type PostForm = z.infer<typeof schema>

const formState = ref<PostForm>({
  title: '',
  content: '',
  category_id: 0,
  published_at: ''
})

const errors = ref<{ [key: string]: string | undefined }>({})
const isSubmitting = ref(false)

const { data: categories } = await useFetch<Category[]>(`${config.public.apiBase}/blog/categories`)

const { data: postData, error: fetchError } = await useFetch<BlogPost>(
    `${config.public.apiBase}/blog/posts/${postId}`
)

watch(postData, (data) => {
  if (data) {
    formState.value = {
      title: data.title,
      content: data.content_raw ?? '',
      category_id: data.category_id ?? 0,
      published_at: data.published_at ?? ''
    }
  }
}, { immediate: true })

if (fetchError.value) {
  toast.add({
    title: 'Помилка',
    description: 'Не вдалося завантажити пост',
    color: 'red'
  })
  router.push('/BlogPostUi')
}

const onSubmit = async () => {
  errors.value = {}

  try {
    schema.parse(formState.value)
  } catch (err: any) {
    if (err instanceof z.ZodError) {
      err.errors.forEach((e: any) => {
        errors.value[e.path[0]] = e.message
      })
      return
    }
  }

  isSubmitting.value = true

  try {
    await $fetch(`${config.public.apiBase}/blog/posts/${postId}`, {
      method: 'PUT',
      body: {
        ...formState.value,
        content_raw: formState.value.content
      }
    })

    toast.add({
      title: 'Успіх',
      description: 'Пост оновлено',
      color: 'green'
    })

    router.push('/admin/blog/posts')
  } catch (err) {
    toast.add({
      title: 'Помилка',
      description: 'Не вдалося оновити пост',
      color: 'red'
    })
  } finally {
    isSubmitting.value = false
  }
}
</script>

<template>
  <div class="relative w-full min-h-screen bg-gradient-to-br from-[#0f0c29] via-[#302b63] to-[#24243e] px-4 py-16 text-white">
    <div class="max-w-2xl mx-auto bg-white/5 p-6 rounded-xl shadow-xl space-y-6 backdrop-blur-md border border-white/10">
      <h1 class="text-3xl font-bold text-white text-center">Редагувати пост</h1>

      <UForm :state="formState" class="space-y-4" @submit="onSubmit">
        <UFormGroup label="Заголовок" :error="errors.title">
          <UInput
              v-model="formState.title"
              placeholder="Введіть заголовок поста"
          />
        </UFormGroup>

        <UFormGroup label="Контент" :error="errors.content">
          <UTextarea
              v-model="formState.content"
              placeholder="Введіть контент поста"
              rows="8"
          />
        </UFormGroup>

        <UFormGroup label="Категорія" :error="errors.category_id">
          <USelect
              v-model="formState.category_id"
              :options="categories?.map(c => ({ label: c.title, value: c.id }))"
              placeholder="Оберіть категорію"
          />
        </UFormGroup>

        <UFormGroup label="Дата публікації (необов'язково)" :error="errors.published_at">
          <UInput
              v-model="formState.published_at"
              type="datetime-local"
          />
        </UFormGroup>

        <div class="flex justify-between pt-4">
          <UButton
              color="gray"
              variant="outline"
              @click="router.push('/BlogPostsUi')"
          >
            ← Назад
          </UButton>

          <UButton
              type="submit"
              color="primary"
              :loading="isSubmitting"
              :disabled="isSubmitting"
          >
            {{ isSubmitting ? 'Зберігається...' : 'Зберегти зміни' }}
          </UButton>
        </div>
      </UForm>
    </div>
  </div>
</template>