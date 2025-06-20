<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-center">
      <div class="w-full">
        <nav class="navbar bg-gray-100 p-4 mb-4 rounded">
          <a href="/" class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
            Головна сторінка
          </a>
        </nav>

        <div class="card bg-white shadow-lg rounded-lg">
          <div class="card-body p-6">
            <div v-if="loading" class="text-center">
              Завантаження...
            </div>

            <div v-else-if="error" class="text-red-500 text-center">
              Помилка: {{ error }}
            </div>

            <table v-else class="table table-auto w-full border-collapse">
              <thead>
              <tr class="bg-gray-50">
                <th class="border p-3 text-left">#</th>
                <th class="border p-3 text-left">Автор</th>
                <th class="border p-3 text-left">Категорія</th>
                <th class="border p-3 text-left">Заголовок</th>
                <th class="border p-3 text-left">Дата публікації</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="post in posts" :key="post.id" class="hover:bg-gray-50">
                <td class="border p-3">{{ post.id }}</td>
                <td class="border p-3">{{ post.user?.name || 'Невідомо' }}</td>
                <td class="border p-3">{{ post.category?.title || 'Без категорії' }}</td>
                <td class="border p-3">
                  <a :href="'/posts/' + post.id "
                     class="text-blue-600 hover:text-blue-800 underline">
                    {{ post.title }}
                  </a>
                </td>
                <td class="border p-3">{{ formatDate(post.published_at) }}</td>
              </tr>
              </tbody>
            </table>

            <div v-if="!loading && posts.length === 0" class="text-center text-gray-500 mt-4">
              Немає постів для відображення
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface User {
  id: number;
  name: string;
}

interface Category {
  id: number;
  title: string;
}

interface Post {
  id: number;
  title: string;
  published_at: string;
  user: User;
  category: Category;
}

const posts = ref<Post[]>([]);
const loading = ref(true);
const error = ref<string | null>(null);

const formatDate = (dateString: string) => {
  if (!dateString) return 'Не опубліковано';
  return new Date(dateString).toLocaleDateString('uk-UA');
};

const getPosts = async () => {
  try {
    loading.value = true;
    error.value = null;

    // Змініть URL на адресу вашого Laravel API
    const response = await $fetch<{data: Post[]}>('http://localhost/api/blog/posts');
    posts.value = response.data;
  } catch (err) {
    console.error('Помилка завантаження постів:', err);
    error.value = 'Не вдалося завантажити пости';
  } finally {
    loading.value = false;
  }
};

// Завантажуємо дані при створенні компонента
onMounted(() => {
  getPosts();
});
</script>