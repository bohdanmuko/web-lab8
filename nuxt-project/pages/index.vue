<script setup lang="ts">
import { h, resolveComponent } from 'vue'
import { useHead } from '#imports'

useHead({
  title: 'Головна сторінка'
})

// Розв'язуємо компоненти Nuxt UI, які будемо використовувати
const UButton = resolveComponent('UButton')
const UCard = resolveComponent('UCard')
</script>

<template>
  <div class="flex-1 w-full h-screen flex items-center justify-center divide-y divide-(--ui-border-accented)">
    <!-- UCard з вирівнюванням тексту по центру -->
    <UCard class="w-96 p-6 text-center">
      <h1 class="text-2xl font-semibold mb-4">Головна сторінка</h1>
      <!-- Кнопка із доданим фоном, градієнтним ефектом та анімацією -->
      <NuxtLink to="/products">
        <UButton
            variant="flat"
            class="rounded-lg py-3 px-5 shadow-md transition-transform transform hover:scale-105 bg-gradient-to-r from-blue-500 to-indigo-500 hover:from-blue-600 hover:to-indigo-600 text-white"
        >
          Перейти до списку продуктів
        </UButton>
      </NuxtLink>
    </UCard>
  </div>
</template>
