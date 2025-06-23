<script setup lang="ts">
import { ref } from 'vue'
import { useRouter, useRuntimeConfig } from '#app'
import { z } from 'zod'
import { useToast } from '#imports'

// Category type interface
interface Category {
  id: number
  title: string
  slug: string
  description: string | null
  parent_id: number | null
}

// Validation schema
const schema = z.object({
  title: z.string().min(3, 'Заголовок має містити щонайменше 3 символи'),
  slug: z.string().optional(),
  description: z.string().optional(),
  parent_id: z.coerce.number().nullable().optional()
})

type CategoryForm = z.infer<typeof schema>

// Main variables
const config = useRuntimeConfig()
const router = useRouter()
const toast = useToast()

const formState = ref<CategoryForm>({
  title: '',
  slug: '',
  description: '',
  parent_id: null
})

const errors = ref<{ [key: string]: string | undefined }>({})
const isSubmitting = ref(false)

// All categories for parent selection
const { data: categories } = await useFetch<Category[]>(`${config.public.apiBase}/blog/categories`)
const parentOptions = ref<Category[]>(categories.value || [])

// Submit handler
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
    await $fetch(`${config.public.apiBase}/blog/categories`, {
      method: 'POST',
      body: formState.value
    })

    toast.add({
      title: 'Успіх',
      description: 'Категорію створено.',
      color: 'green'
    })

    router.push('/BlogCategories')
  } catch (error) {
    toast.add({
      title: 'Помилка',
      description: 'Не вдалося створити категорію.',
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
      <h1 class="text-3xl font-bold text-white text-center">Створити категорію</h1>

      <UForm :state="formState" class="space-y-4" @submit="onSubmit">
        <UFormGroup label="Назва категорії" :error="errors.title">
          <UInput v-model="formState.title" />
        </UFormGroup>

        <UFormGroup label="Псевдонім (slug)" :error="errors.slug">
          <UInput v-model="formState.slug" />
        </UFormGroup>

        <UFormGroup label="Опис" :error="errors.description">
          <UTextarea v-model="formState.description" />
        </UFormGroup>

        <UFormGroup label="Батьківська категорія" :error="errors.parent_id">
          <USelect
              v-model="formState.parent_id"
              :options="parentOptions.map(cat => ({ label: cat.title, value: cat.id }))"
              placeholder="Оберіть батьківську категорію"
          />
        </UFormGroup>

        <div class="flex justify-between">
          <UButton color="gray" @click="router.push('/BlogCategories')">
            ← Назад
          </UButton>

          <UButton type="submit" color="primary" :loading="isSubmitting">
            Створити
          </UButton>
        </div>
      </UForm>
    </div>
  </div>
</template>