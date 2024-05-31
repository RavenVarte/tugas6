<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <q-page padding>
        <q-form @submit="save" class="q-gutter-md">
          <q-input filled v-model="form.title" label="Title" />
          <q-input filled type="textarea" v-model="form.content" label="Content" />
          <q-btn type="submit" label="Save" color="primary" />
        </q-form>
        
        <q-list bordered class="q-mt-md">
          <q-item v-for="article in articles" :key="article.id">
            <q-item-section>
              <q-item-label>{{ article.title }}</q-item-label>
              <q-item-label caption>{{ article.content }}</q-item-label>
            </q-item-section>
            <q-item-section side>
              <q-btn flat round icon="edit" @click="edit(article)" />
              <q-btn flat round icon="delete" color="negative" @click="remove(article.id)" />
            </q-item-section>
          </q-item>
        </q-list>

        <q-btn label="Load" @click="load" color="secondary" class="q-mt-md" />

        <q-footer class="text-center q-pa-md">
          <div>&copy; 2024 Integrasi Dengan API. All rights reserved.</div>
          <div>Contact Person: ebbyarrahmantarif@gmail.com</div>
        </q-footer>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';
import axios from 'axios';
import { QLayout, QPage, QPageContainer, QForm, QInput, QBtn, QList, QItem, QItemSection, QItemLabel, QFooter } from 'quasar';

const API_KEY = '$2a$10$5XM6y/4VUvVGNzBsIU2GGelF2gQ0K0pdFBQypYZCJe9VPXjMOfDu6';
const BIN_ID = '6659fdc8e41b4d34e4fc85b8';
const BIN_URL = `https://api.jsonbin.io/v3/b/${BIN_ID}`;
const HEADERS = {
  'X-Master-Key': API_KEY,
  'Content-Type': 'application/json',
};

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get(`${BIN_URL}/latest`, { headers: HEADERS });
    // Ensure the response data is in the correct format
    if (Array.isArray(response.data.record)) {
      articles.value = response.data.record;
    } else {
      articles.value = [];
      console.error('Data loaded is not an array', response.data);
    }
  } catch (error) {
    console.error('Error loading articles', error);
  }
}

async function save() {
  if (form.id) {
    await update();
  } else {
    await create();
  }
  resetForm();
}

async function create() {
  try {
    const newArticle = { id: Date.now(), title: form.title, content: form.content };
    articles.value.push(newArticle);
    await updateBin();
  } catch (error) {
    console.error('Error creating article', error);
  }
}

async function update() {
  try {
    const index = articles.value.findIndex(article => article.id === form.id);
    articles.value[index] = { id: form.id, title: form.title, content: form.content };
    await updateBin();
  } catch (error) {
    console.error('Error updating article', error);
  }
}

async function remove(id) {
  try {
    articles.value = articles.value.filter(article => article.id !== id);
    await updateBin();
  } catch (error) {
    console.error('Error deleting article', error);
  }
}

async function updateBin() {
  try {
    await axios.put(BIN_URL, { record: articles.value }, { headers: HEADERS });
  } catch (error) {
    console.error('Error updating bin', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

function resetForm() {
  form.id = null;
  form.title = '';
  form.content = '';
}

onMounted(load);
</script>

<style>
.q-page {
  max-width: 600px;
  margin: 0 auto;
}

.q-footer {
  margin-top: 2rem;
}
</style>
