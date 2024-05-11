<template>
  <div class="centered-container">
    <div class="bordered-container">
      <h1>{{ pageTitle }}</h1>
      <div class="header-menu">
        <template v-if="currentView !== null">
          <button @click="backToHome" class="back-btn">Kembali</button>
        </template>
        <button @click="showTodos" class="todos-btn">Todos</button>
        <button @click="showPosts" class="posts-btn">Post</button>
      </div>
      <div v-if="currentView === 'todos'">
        <div class="input-container">
          <div class="input-group">
            <input type="text" v-model.trim="newTask" @keyup.enter="addTask" placeholder="Tambahkan daftar kegiatan">
            <button @click="addTask" class="add-btn">Tambah</button>
          </div>
        </div>
        <button class="show-incompleted-btn" @click="toggleShowCompleted">
          {{ showCompleted ? 'Keluar ' : 'Tampilkan yang belum Selesai ' }}
        </button>
        <div v-if="tasks.length > 0" class="task-table">
          <thead>
            <tr>
              <th>NO.</th>
              <th>KEGIATAN</th>
              <th>KETERANGAN</th>
              <th>METODE</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(task, index) in filteredTasks" :key="index" :class="{ 'completed-task': task.completed }">
              <td>{{ index + 1 }}</td>
              <td class="task-name">{{ task.name }}</td>
              <td>{{ task.completed ? 'Selesai' : 'Belum Selesai' }}</td>
              <td>
                <button @click="toggleTaskCompletion(task)" class="action-btn" title="Selesai">
                  <i v-if="!task.completed" class="fas fa-check">Selesaikan</i>
                  <i v-else class="fas fa-times">Batalkan</i>
                </button>
                <button @click="removeTask(task)" class="action-btn" title="Hapus">
                  <i class="fas fa-trash-alt">Hapus</i>
                </button>
              </td>
            </tr>
          </tbody>
        </div>
        <p v-else class="no-tasks">Silahkan masukan to do list anda.</p>
      </div>

      <div v-if="currentView === 'posts'">
        <div class="input-container">
          <select v-model="selectedUser">
            <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</option>
          </select>
        </div>
        <div v-if="selectedUser !== null">
          <table v-if="posts.length > 0" class="post-table">
            <thead>
              <tr>
                <th>NO.</th>
                <th>JUDUL</th>
                <th>ISI POST</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(post, index) in filteredPosts" :key="index">
                <td>{{ index + 1 }}</td>
                <td>{{ post.title }}</td>
                <td>{{ post.body }}</td>
              </tr>
            </tbody>
          </table>
          <p v-else class="no-posts">Tidak ada postingan untuk pengguna ini.</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';

export default {
  setup() {
    const currentView = ref(null);
    const pageTitle = ref('');

    const newTask = ref('');
    const tasks = ref(JSON.parse(localStorage.getItem('tasks')) || []);
    const showCompleted = ref(JSON.parse(localStorage.getItem('showCompleted')) || false);
    const taskCounter = ref(tasks.value.length + 1);

    const saveTasksToLocalStorage = () => {
      localStorage.setItem('tasks', JSON.stringify(tasks.value));
    };

    const saveShowCompletedToLocalStorage = () => {
      localStorage.setItem('showCompleted', JSON.stringify(showCompleted.value));
    };

    const addTask = () => {
      const trimmedTask = newTask.value.trim();
      if (trimmedTask !== '' && !tasks.value.find(task => task.name === trimmedTask)) {
        tasks.value.push({ name: trimmedTask, completed: false });
        newTask.value = '';
        taskCounter.value++;
        saveTasksToLocalStorage();
      }
    };

    const removeTask = (task) => {
      const index = tasks.value.findIndex(t => t.name === task.name);
      tasks.value.splice(index, 1);
      saveTasksToLocalStorage();
    };

    const toggleTaskCompletion = (task) => {
      task.completed = !task.completed;
      saveTasksToLocalStorage();
    };

    const toggleShowCompleted = () => {
      showCompleted.value = !showCompleted.value;
      saveShowCompletedToLocalStorage();
    };

    const filteredTasks = computed(() => {
      if (showCompleted.value) {
        return tasks.value.filter(task => !task.completed);
      } else {
        return tasks.value;
      }
    });

    const selectedUser = ref(null);
    const users = ref([]);

    const loadUsers = async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      users.value = await response.json();
    };

    const posts = ref([]);
    const filteredPosts = computed(() => {
      return posts.value.filter(post => post.userId === selectedUser.value);
    });

    const loadPosts = async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts');
      posts.value = await response.json();
    };

    onMounted(() => {
      loadUsers();
      loadPosts();
    });

    const showTodos = () => {
      currentView.value = 'todos';
      pageTitle.value = '';
    };

    const showPosts = () => {
      currentView.value = 'posts';
      pageTitle.value = 'Silahkan pilih salah satu user';
    };

    const backToHome = () => {
      currentView.value = null;
      pageTitle.value = '';
    };

    return {
      currentView,
      pageTitle,
      newTask,
      tasks,
      showCompleted,
      addTask,
      removeTask,
      toggleTaskCompletion,
      toggleShowCompleted,
      filteredTasks,
      selectedUser,
      users,
      posts,
      filteredPosts,
      showTodos,
      showPosts,
      backToHome
    };
  }
};
</script>


