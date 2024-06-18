<template>
  <div class="full-background">
    <div class="top-left-label">M.TAUFIK KURRAHMAN <br>TUGAS PERTEMUAN 3</div>

    <div class="centered-container">
      <div class="bordered-container">
        <div class="header-menu">
          <h1>{{ pageTitle }}</h1>
        </div>
        <div class="input-container">
          <div class="input-group">
            <input type="text" v-model.trim="newTask" @keyup.enter="addTask" placeholder="Tambahkan daftar kegiatan">
            <button @click="addTask" class="add-btn">Tambah</button>
          </div>
        </div>
        <button class="show-incompleted-btn" @click="toggleShowCompleted">
          {{ showCompleted ? 'Tampilkan Semua' : 'Tampilkan yang belum Selesai' }}
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
            <tr v-for="(task, index) in filteredTasks" :key="index" :class="{ 'completed-task': task.completed, 'editing-task': task.editing }">
              <td>{{ index + 1 }}</td>
              <td class="task-name">
                <span v-if="!task.editing">{{ task.name }}</span>
                <input v-else type="text" v-model="task.name" @blur="stopEditing(task)" @keyup.enter="stopEditing(task)" class="edit-input">
              </td>
              <td>{{ task.completed ? 'Selesai' : 'Belum Selesai' }}</td>
              <td>
                <button @click="toggleTaskCompletion(task)" class="action-btn" title="Selesai">
                  <i v-if="!task.completed" class="fas fa-check"> Selesaikan</i>
                  <i v-else class="fas fa-times"> Batalkan</i>
                </button>
                <button @click="removeTask(task)" class="action-btn" title="Hapus">
                  <i class="fas fa-trash-alt"> Hapus</i>
                </button>
                <button @click="editTask(task)" class="action-btn" title="Edit">
                  <i class="fas fa-edit"> Edit</i>
                </button>
              </td>
            </tr>
          </tbody>
        </div>
        <p v-else class="no-tasks">Silahkan masukan to do list anda.</p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    const pageTitle = ref('Daftar Kegiatan');
    const newTask = ref('');
    const tasks = ref([
      { name: 'Mengerjakan tugas akhir PBK', completed: true },
      { name: 'Belajar untuk ujian', completed: true },
      { name: 'Lulus kuliah usia 22', completed: false },
    ]);
    const showCompleted = ref(false);

    const saveTasksToLocalStorage = () => {
      localStorage.setItem('tasks', JSON.stringify(tasks.value));
    };

    const saveShowCompletedToLocalStorage = () => {
      localStorage.setItem('showCompleted', JSON.stringify(showCompleted.value));
    };

    const addTask = () => {
      const trimmedTask = newTask.value.trim();
      if (trimmedTask !== '' && !tasks.value.find(task => task.name === trimmedTask)) {
        tasks.value.push({ name: trimmedTask, completed: false, editing: false });
        newTask.value = '';
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

    const editTask = (task) => {
      task.editing = true;
    };

    const stopEditing = (task) => {
      task.editing = false;
      saveTasksToLocalStorage();
    };

    return {
      pageTitle,
      newTask,
      tasks,
      showCompleted,
      addTask,
      removeTask,
      toggleTaskCompletion,
      toggleShowCompleted,
      filteredTasks,
      editTask,
      stopEditing
    };
  }
};
</script>

<style scoped>
.full-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #74ebd5, #9face6);
  font-family: 'Roboto', sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: -1; /* Push behind other content */
}

.top-left-label {
  position: absolute;
  top: 20px;
  left: 20px;
  font-size: 18px;
  font-weight: 700;
  color: #ffffff;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5); /* Tambahkan shadow untuk teks */
}

.centered-container {
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1; /* Ensure content is above the background */
}

.bordered-container {
  border: 2px solid #007bff;
  padding: 20px;
  border-radius: 15px;
  width: 80%;
  max-width: 800px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
  background: white;
  animation: fadeIn 1s ease-in-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.header-menu {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.header-menu h1 {
  font-size: 24px;
  font-weight: 500;
  margin: 0;
}

.header-menu button {
  padding: 10px 20px;
  cursor: pointer;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}

.header-menu button:hover {
  background-color: #0056b3;
}

.input-container {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.input-group {
  display: flex;
  width: 100%;
  max-width: 600px;
}

.input-group input {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px 0 0 5px;
  outline: none;
  transition: border-color 0.3s ease;
}

.input-group input:focus {
  border-color: #007bff;
}

.input-group button {
  padding: 10px 20px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 0 5px 5px 0;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.input-group button:hover {
  background-color: #0056b3;
}

.show-incompleted-btn {
  margin-bottom: 20px;
  padding: 10px 20px;
  cursor: pointer;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}

.show-incompleted-btn:hover {
  background-color: #0056b3;
}

.task-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.task-table th,
.task-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: left;
  transition: background-color 0.3s ease;
}

.task-table th {
  background-color: #f1f1f1;
}

.completed-task td {
  text-decoration: line-through;
  background-color: #d4edda;
}

.editing-task td {
  background-color: #e9f7fd;
}

.action-btn {
  background: none;
  border: none;
  cursor:
  pointer;
  margin: 0 5px;
  transition: color 0.3s ease;
}

.action-btn:hover {
  color: #007bff;
}

.no-tasks {
  text-align: center;
  color: #666;
  margin-top: 20px;
  font-size: 18px;
}

.edit-input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  outline: none;
  transition: border-color 0.3s ease;
}

.edit-input:focus {
  border-color: #007bff;
  background-color: #f0f8ff;
}
</style>
