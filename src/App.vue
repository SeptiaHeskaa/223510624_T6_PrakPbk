<template>
  <div>
    <form @submit.prevent="handleSubmit" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="input"></textarea><br />
      <button type="submit" class="btn">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <strong>{{ article.title }}</strong><br />
          {{ article.content }}
        </div>
        <div class="article-actions">
          <button @click="handleEdit(article)" class="btn btn-edit">Edit</button>
          <button @click="handleDelete(article.id)" class="btn btn-delete">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="handleLoad" class="btn load-btn">Load Articles</button>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    const loadArticles = async () => {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error.message);
      }
    };

    const saveArticle = async () => {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios({ url, method, data: form });
        
        if (form.id) {
          const index = articles.value.findIndex(article => article.id === form.id);
          articles.value[index] = response.data;
        } else {
          articles.value.push(response.data);
        }
        
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error.message);
      }
    };

    const handleEdit = (article) => {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    };

    const deleteArticle = async (id) => {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error.message);
      }
    };

    const handleSubmit = () => {
      saveArticle();
    };

    const handleLoad = () => {
      loadArticles();
    };

    const handleDelete = (id) => {
      deleteArticle(id);
    };

    onMounted(loadArticles);

    return {
      form,
      articles,
      handleSubmit,
      handleLoad,
      handleEdit,
      handleDelete,
    };
  },
};
</script>

<style>
/* CSS Styling */
body {
  font-family: 'Times New Roman', Times, serif;
  margin: 0;
  padding: 0;
}

.form {
  margin-bottom: 24px;
}

.input {
  width: 100%;
  padding: 12px;
  margin-bottom: 10px;
  border: 1px solid #5d0a0a;
  border-radius: 5px;
  color: rgb(208, 98, 98);
}

.btn {
  padding: 10px 16px;
  background-color: #b90a62;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn:hover {
  background-color: #1a020e;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #ea00a0;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 10px;
}

.article-content {
  margin-bottom: 10px;
  color: black;
}

.article-actions {
  text-align: right;
}

.btn-edit {
  background-color: #ba19bd;
  margin-right: 5px;
}

.btn-delete {
  background-color: #ef2bc4;
}

.load-btn {
  background-color: #ff007b;
  margin-top: 20px;
}
</style>
