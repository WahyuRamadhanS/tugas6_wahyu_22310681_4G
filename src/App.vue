<script setup>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get('http://localhost:3000/articles');
    articles.value = response.data;
  } catch (error) {
    console.error("Error loading articles: ", error);
  }
}

async function save() {
  try {
    const url = form.id
      ? `http://localhost:3000/articles/${form.id}`
      : 'http://localhost:3000/articles';
    const method = form.id ? 'put' : 'post';
    const response = await axios[method](url, form);

    if (form.id) {
      const index = articles.value.findIndex(article => article.id === form.id);
      if (index !== -1) {
        articles.value[index] = response.data;
      }
    } else {
      articles.value.push(response.data);
    }

    form.id = null;
    form.title = '';
    form.content = '';
  } catch (error) {
    console.error('Error saving article: ', error);
  }
}

async function remove(id) {
  try {
    await axios.delete(`http://localhost:3000/articles/${id}`);
    articles.value = articles.value.filter(article => article.id !== id);
  } catch (error) {
    console.error('Error deleting article: ', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

onMounted(load);
</script>

<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3 class="article-title">{{ article.title }}</h3>
        <p class="article-content">{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="remove(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<style scoped>
.container {
  background-color: #C4D7B2;
  padding: 2rem;
  border-radius: 8px;
  max-width: 600px;
  margin: auto;
  padding: 20px;
}

.form {
  margin-bottom: 2rem;
  padding: 20px;
}

.input, .textarea {
  width: 100%;
  padding: 0.5rem;
  margin-bottom: 1rem;
  border: 1px solid #A0C49D;
  border-radius: 4px;
}

.textarea {
  height: 100px;
}

.button {
  background-color: #77ac72;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 0.5rem;
  box-shadow: rgba(0, 0, 0, 0.619);
}

.button:hover {
  background-color: #88A985;
  box-shadow: rgba(0, 0, 0, 0.619);
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  background-color: white;
  padding: 1rem;
  margin-bottom: 1rem;
  border: 1px solid #A0C49D;
  border-radius: 4px;
}

.article-title {
  margin: 0 0 0.5rem 0;
}

.article-content {
  margin: 0 0 1rem 0;
}

.edit-button {
  background-color: #77ac72;
  box-shadow: rgba(0, 0, 0, 0.619);
}

.delete-button {
  background-color: #D9534F;
}

.load-button {
  display: block;
  margin: 1rem auto;
}

</style>
