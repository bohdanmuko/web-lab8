<template>
  <div>
    <h1 class="text-3xl font-bold text-center mb-8">Блог пости</h1>
    <PostsTableComponent />
  </div>
</template>

<script setup lang="ts">
// Метадані сторінки
useHead({
  title: 'Блог пости',
  meta: [
    { name: 'description', content: 'Список всіх постів блогу' }
  ]
});
</script>