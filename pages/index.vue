<template>
  <div class="min-h-screen flex flex-col items-center justify-center space-y-6 p-4">
    <h1 class="text-3xl font-bold mb-4">👋 Виберіть розділ</h1>

    <div class="flex gap-4">
      <UButton to="/products" color="emerald" size="lg" icon="i-heroicons-cube-transparent">
        Таблиця продуктів
      </UButton>

      <UButton to="/BlogPosts" color="emerald" size="lg" icon="i-heroicons-newspaper">
        Таблиця постів
      </UButton>
      <UButton to="/BlogPostsUi" label="Пости через UTable" color="emerald" />
    </div>
  </div>
</template>

<script setup>

</script>
