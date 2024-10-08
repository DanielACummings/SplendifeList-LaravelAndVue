<script setup>
import axios from 'axios';
import { Head } from '@inertiajs/vue3';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { ref, onMounted } from 'vue';

const lists = ref([]);
const newListName = ref('');
const newItemName = ref('');

onMounted(() => {
  loadLists();
});

function loadLists() {
  axios.get('item-lists')
    .then(response => {
      lists.value = response.data;
      lists.value.forEach(list => getItemsByList(list.id));
    })
    .catch(error => {
      console.error(error);
    });
}

function createList() {
  if (newListName.value.trim() === '') {
    alert('Please enter a list name');
    return;
  }
  if (lists.value.find(list => list.name === newListName.value)) {
    alert('List name already in use');
    return;
  }

  // TODO: Remove hard-coded test user ID once authentication code is added
  axios.post('/item-lists', { name: newListName.value, user: 1 })
    .then(() => {
      loadLists();
      newListName.value = '';
    })
    .catch(error => {
      console.error(error);
    });
}

function getItemsByList(listId) {
  // TODO: Remove hard-coded test user ID once authentication code is added
  axios.get(`lists/${listId}/items?userId=1`)
    .then(response => {
      const list = lists.value.find(list => list.id === listId);
      list.items = response.data;
    })
    .catch(error => {
      console.error(error);
    });
}

function addItem(listId) {
  if (newItemName.value.trim() === '') {
    alert('Please enter a list item name');
    return;
  }

  axios.post(`lists/${listId}/items`, { name: newItemName.value })
    .then(() => {
      loadLists();
      newItemName.value = '';
    })
    .catch(error => {
      console.error(error);
    });
}
</script>

<template>
    <Head title="Dashboard" />

    <AuthenticatedLayout>
        <template #header>
            <h2
                class="text-xl font-semibold leading-tight text-gray-800 dark:text-gray-200"
            >
                Dashboard
            </h2>
        </template>

        <div class="py-12">
            <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
                <div
                    class="overflow-hidden bg-white shadow-sm sm:rounded-lg dark:bg-gray-800"
                >
                    <div class="p-6 text-gray-900 dark:text-gray-100">
                        You're logged in!
                    </div>
                </div>
            </div>
        </div>
        <div>
            <h1>Lists</h1>
            <ul>
              <li v-for="list in lists" :key="list.id">
                <span :class="{ 'crossed-out': list.crossedOut }">{{ list.name }}</span>
                <ul>
                  <li v-for="item in list.items" :key="item.id">
                    <span :class="{ 'crossed-out': item.crossedOut }">{{ item.name }}</span>
                  </li>
                </ul>
              </li>
            </ul>
            <form @submit.prevent="createList">
              <input type="text" v-model="newListName" placeholder="Enter new list name">
              <button type="submit">Create List</button>
            </form>
          </div>
    </AuthenticatedLayout>
</template>
