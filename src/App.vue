<template>
  <div>
    <!-- Form untuk menambah atau mengedit artikel -->
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">{{ form.id ? 'Update' : 'Save' }}</button>
      <button type="button" @click="clearForm">Cancel</button>
    </form>

    <!-- Daftar artikel -->
    <ul>
      <li v-for="article in articles" :key="article.id">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <div class="button-group">
          <button @click="edit(article)">Edit</button>
          <button @click="deleteArticle(article.id)">Delete</button>
        </div>
      </li>
    </ul>

    <!-- Tombol untuk memuat artikel -->
    <button @click="load">Load Articles</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    // State untuk form artikel
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    // State untuk daftar artikel
    const articles = ref([]);

    // Fungsi untuk memuat artikel dari server
    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    // Fungsi untuk menyimpan atau mengupdate artikel
    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          // Update artikel yang sudah ada
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Tambah artikel baru
          articles.value.push(response.data);
        }

        clearForm(); // Bersihkan form setelah menyimpan
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    // Fungsi untuk menghapus artikel
    async function deleteArticle(articleId) {
      try {
        await axios.delete(`http://localhost:3000/articles/${articleId}`);
        // Hapus artikel dari daftar
        articles.value = articles.value.filter((article) => article.id !== articleId);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    // Fungsi untuk mengisi form saat tombol edit diklik
    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    // Fungsi untuk membersihkan form
    function clearForm() {
      form.id = null;
      form.title = '';
      form.content = '';
    }

    // Panggil load saat komponen dimount
    onMounted(load);

    // Ekspor data dan fungsi yang diperlukan ke template
    return { form, articles, save, edit, deleteArticle, clearForm };
  },
};
</script>

<style scoped>
.button-group {
  margin-top: 10px;
}

.button-group button {
  margin-right: 5px;
}
</style>
