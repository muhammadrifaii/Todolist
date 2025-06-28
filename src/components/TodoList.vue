<template>
  <div class="todo-app">
    <h1>Todo List dengan Axios & JSON Server (CRUD lengkap)</h1>

    <!-- Form tambah todo baru -->
    <form @submit.prevent="addTodo">
      <input v-model="newTodo" placeholder="Tambah todo baru..." required />
      <button type="submit" :disabled="loading">Tambah</button>
    </form>

    <p v-if="loading">Memuat data...</p>
    <p v-if="error" style="color: red;">{{ error }}</p>

    <ul v-if="todos.length">
      <li v-for="todo in todos" :key="todo.id">
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleCompleted(todo)"
        />

        <!-- Tampilan normal -->
        <template v-if="editId !== todo.id">
          <span :style="{ textDecoration: todo.completed ? 'line-through' : 'none' }">
            {{ todo.title }}
          </span>
          <button @click="startEdit(todo)">Edit</button>
        </template>

        <!-- Form edit -->
        <template v-else>
          <input v-model="editTitle" />
          <button @click="updateTodo(todo)">Simpan</button>
          <button @click="cancelEdit">Batal</button>
        </template>

        <button @click="deleteTodo(todo.id)">Hapus</button>
      </li>
    </ul>

    <p v-else>Tidak ada todo.</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const todos = ref([]);
const newTodo = ref('');
const loading = ref(false);
const error = ref('');

const editId = ref(null);
const editTitle = ref('');

const API_URL = 'http://localhost:3000/todos';

const loadTodos = async () => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.get(API_URL);
    todos.value = response.data;
  } catch (err) {
    error.value = 'Gagal memuat data todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const addTodo = async () => {
  if (!newTodo.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const newData = {
      title: newTodo.value,
      completed: false,
    };
    const response = await axios.post(API_URL, newData);
    todos.value.push(response.data);
    newTodo.value = '';
  } catch (err) {
    error.value = 'Gagal menambah todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const toggleCompleted = async (todo) => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      completed: !todo.completed,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const deleteTodo = async (id) => {
  loading.value = true;
  error.value = '';
  try {
    await axios.delete(`${API_URL}/${id}`);
    todos.value = todos.value.filter(todo => todo.id !== id);
  } catch (err) {
    error.value = 'Gagal menghapus todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const startEdit = (todo) => {
  editId.value = todo.id;
  editTitle.value = todo.title;
};

const cancelEdit = () => {
  editId.value = null;
  editTitle.value = '';
};

const updateTodo = async (todo) => {
  if (!editTitle.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      title: editTitle.value,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
    editId.value = null;
    editTitle.value = '';
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

onMounted(loadTodos);
</script>

<style scoped>
.todo-app {
  max-width: 600px;
  margin: 40px auto;
  padding: 20px;
  background-color: #f7f9fc;
  border-radius: 12px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
  font-family: 'Segoe UI', sans-serif;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input[type="text"],
input {
  flex: 1;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  transition: 0.3s;
}
input:focus {
  border-color: #007bff;
  outline: none;
}

button {
  padding: 10px 14px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}
button:hover {
  background-color: #0056b3;
}
button:disabled {
  background-color: #aaa;
  cursor: not-allowed;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  display: flex;
  align-items: center;
  background: white;
  padding: 12px 16px;
  border-radius: 10px;
  margin-bottom: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  transition: 0.2s ease;
}
li:hover {
  transform: scale(1.01);
}

li span {
  flex-grow: 1;
  margin-left: 12px;
  font-size: 16px;
  color: #333;
}

li input[type="checkbox"] {
  transform: scale(1.2);
}

li input[type="text"] {
  flex-grow: 1;
  margin-left: 12px;
}

li button {
  margin-left: 8px;
  font-size: 13px;
}

li button:nth-of-type(1) {
  background-color: #28a745; /* Hijau untuk simpan/edit */
}
li button:nth-of-type(1):hover {
  background-color: #1e7e34;
}
li button:nth-of-type(2) {
  background-color: #ffc107; /* Kuning untuk batal */
}
li button:nth-of-type(2):hover {
  background-color: #d39e00;
}
li button:last-of-type {
  background-color: #dc3545; /* Merah untuk hapus */
}
li button:last-of-type:hover {
  background-color: #b02a37;
}
</style>
