<template>
  <div class="container">
    <h2>Quản lý công việc</h2>
    <div class="add-task">
      <input
        type="text"
        placeholder="Nhập tên công việc"
        v-model="newTask.name"
      />
      <button @click="addOrUpdateTask">
        {{ currentTaskId ? "Cập nhật" : "Thêm" }}
      </button>
    </div>
    <div class="filters">
      <button :class="{ active: filter === 'all' }" @click="filter = 'all'">
        Tất cả
      </button>
      <button
        :class="{ active: filter === 'completed' }"
        @click="filter = 'completed'"
      >
        Hoàn thành
      </button>
      <button
        :class="{ active: filter === 'active' }"
        @click="filter = 'active'"
      >
        Đang thực hiện
      </button>
    </div>
    <div class="task-list">
      <div class="task" v-for="task in filteredTasks" :key="task.id">
        <input
          type="checkbox"
          :checked="task.completed"
          @change="toggleTask(task.id)"
        />
        <span :class="{ completed: task.completed }">{{ task.name }}</span>
        <div class="actions">
          <span class="edit" @click="editTask(task.id)">Sửa</span>
          <span class="delete" @click="deleteTask(task.id)">Xóa</span>
        </div>
      </div>
    </div>
    <div class="clear-buttons">
      <button class="clear-completed" @click="clearCompleted">
        Xóa công việc hoàn thành
      </button>
      <button class="clear-all" @click="clearAll">Xóa tất cả công việc</button>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, computed, onMounted } from "vue";

const newTask = ref({ name: "", completed: false });
const tasks = ref([]);
const filter = ref("all");
const currentTaskId = ref(null);

const getAllTask = async () => {
  const res = await axios.get(" http://localhost:8080/tasks");
  tasks.value = res.data;
};

onMounted(() => {
  getAllTask();
});

const filteredTasks = computed(() => {
  switch (filter.value) {
    case "completed":
      return tasks.value.filter((task) => task.completed);
    case "active":
      return tasks.value.filter((task) => !task.completed);
    default:
      return tasks.value;
  }
});

const editTask = async (id) => {
  const task = tasks.value.find((task) => task.id == id);
  if (task) {
    newTask.value = { ...task };
    currentTaskId.value = id;
  }
};

const addOrUpdateTask = async () => {
  if (currentTaskId.value) {
    const isConfirmed = window.confirm(
      "Bạn có chắc chắn muốn chỉnh sửa công việc này không?"
    );
    if (isConfirmed) {
      const updateTask = {
        ...newTask.value,
      };
      await axios.patch(
        `http://localhost:8080/tasks/${currentTaskId.value}`,
        updateTask
      );
      newTask.value = { name: "", completed: false };
      currentTaskId.value = null;
      getAllTask();
      alert("Chỉnh sửa thành công");
    } else {
      newTask.value = { name: "", completed: false };
      currentTaskId.value = null;
    }
  } else {
    if (newTask.value.name.trim()) {
      const createTask = {
        ...newTask.value,
      };
      await axios.post("http://localhost:8080/tasks", createTask);
      alert("Thêm thành công");
      getAllTask();
      newTask.value = { name: "", completed: false };
    }
  }
};

const toggleTask = async (id) => {
  const task = tasks.value.find((task) => task.id == id);
  if (task) {
    const updatedTask = { ...task, completed: !task.completed };
    await axios.patch(`http://localhost:8080/tasks/${id}`, updatedTask);
    getAllTask();
  }
};

const deleteTask = async (id) => {
  const isConfirmed = window.confirm(
    "Bạn có chắc chắn muốn xóa công việc này không?"
  );
  if (isConfirmed) {
    await axios.delete(`http://localhost:8080/tasks/${id}`);
    alert("Xóa công việc thành công!");
    getAllTask();
  }
};

const clearCompleted = async () => {
  const completedTasks = tasks.value.filter((task) => task.completed);
  const deletePromises = completedTasks.map((completedTask) =>
    axios.delete(`http://localhost:8080/tasks/${completedTask.id}`)
  );
  await Promise.all(deletePromises);
  getAllTask();
  alert("Đã xóa tất cả công việc hoàn thành!");
};

const clearAll = async () => {
  const deleteAll = tasks.value.map((task) =>
    axios.delete(`http://localhost:8080/tasks/${task.id}`)
  );
  await Promise.all(deleteAll);
  getAllTask();
  alert("Đã xóa tất cả công việc!");
};
</script>

<style scoped>
.container {
  font-family: Arial, sans-serif;
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

h2 {
  text-align: center;
  color: #333;
}

.add-task {
  display: flex;
  margin-bottom: 20px;
}

input[type="text"] {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

button {
  padding: 10px 15px;
  background-color: #4285f4;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.filters {
  display: flex;
  margin-bottom: 20px;
}

.filters button {
  flex-grow: 1;
  background-color: #f1f3f4;
  color: #333;
  border: 1px solid #ddd;
  border-radius: 0;
}

.filters button.active {
  background-color: #4285f4;
  color: white;
}

.task {
  display: flex;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.task input[type="checkbox"] {
  margin-right: 10px;
}

.task span {
  flex-grow: 1;
}

.task span.completed {
  text-decoration: line-through;
  color: #888;
}

.actions {
  display: flex;
  gap: 10px;
}

.actions .edit {
  cursor: pointer;
  background-color: #ffd900;
  color: white;
  padding: 5px;
}

.actions .delete {
  cursor: pointer;
  background-color: #ff4646;
  color: white;
  padding: 5px;
}

.clear-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.clear-buttons button {
  background-color: #f44336;
  border-radius: 4px;
}
</style>
