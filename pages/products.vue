<script setup lang="ts">
import ProductTable from '~/components/ProductTable.vue'

useHead({
  title: 'Список продуктів'
})
</script>

<template>
  <div class="relative w-full min-h-screen bg-gradient-to-br from-[#0f0c29] via-[#302b63] to-[#24243e] px-4 py-16">
    <div class="max-w-6xl mx-auto flex items-center justify-center h-[120px] mb-10">
      <h1
          class="text-4xl font-bold text-white text-center tracking-tight px-6 py-4 rounded-xl border border-white/20 shadow-lg backdrop-blur-md bg-white/10 flex items-center justify-center"
      >
        Список продуктів
      </h1>
    </div>

    <ProductTable />
  </div>
</template>
