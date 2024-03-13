<template>
  <div id="app">
    <h1>To-Do List</h1>
    <div class="add-task-section">
      <input v-model="newTask" @keyup.enter="addTask" class="add-task-input" placeholder="Add a new task">
      <button @click="addTask" class="add-task-btn">Add Task</button>
    </div>
    <div class="task-section">
      <h2>Open Tasks</h2>
      <ul>
        <li v-for="(task, index) in openTasks" :key="index" class="task">
          <div class="task-info">
            <span class="task-description">{{ task.description }}</span>
            <span class="task-date">Created at: {{ task.createdAt }}</span>
          </div>
          <button @click="completeTask(task.id)" class="complete-btn">Complete</button>
          <button @click="deleteTask(task.id)" class="delete-btn">Delete</button>
        </li>
        <li v-if="openTasks.length === 0" class="no-task">No open tasks</li>
      </ul>
    </div>
    <div class="task-section">
      <h2>Completed Tasks</h2>
      <ul>
        <li v-for="(task, index) in completedTasks" :key="index" class="task">
          <div class="task-info">
            <span class="task-description">{{ task.description }}</span>
            <span class="task-date">Created at: {{ task.createdAt }} - Completed at: {{ task.completedAt }}</span>
          </div>
          <button @click="deleteTask(task.id)" class="delete-btn">Delete</button>
        </li>
        <li v-if="completedTasks.length === 0" class="no-task">No completed tasks</li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      newTask: '',
      tasks: [],
    };
  },
  computed: {
    openTasks() {
      return this.tasks.filter(task => !task.completed);
    },
    completedTasks() {
      return this.tasks.filter(task => task.completed);
    },
  },
  methods: {
    async fetchTasks() {
      try {
        const response = await axios.get('http://localhost:3000/tasks');
        this.tasks = response.data;
      } catch (error) {
        console.error('Error fetching tasks:', error);
      }
    },
    async addTask() {
      if (!this.newTask.trim()) return;
      try {
        const response = await axios.post('http://localhost:3000/tasks', {
          description: this.newTask,
          createdAt: new Date().toLocaleString(),
          completed: false,
        });
        this.tasks.push(response.data);
        this.newTask = '';
      } catch (error) {
        console.error('Error adding task:', error);
      }
    },
    async completeTask(taskId) {
      const task = this.tasks.find(t => t.id === taskId);
      if (!task) return;

      task.completed = true;
      task.completedAt = new Date().toLocaleString();

      try {
        await axios.put(`http://localhost:3000/tasks/${taskId}`, task);
      } catch (error) {
        console.error('Error completing task:', error);
      }
    },
    async deleteTask(taskId) {
      try {
        await axios.delete(`http://localhost:3000/tasks/${taskId}`);
        this.tasks = this.tasks.filter(task => task.id !== taskId);
      } catch (error) {
        console.error('Error deleting task:', error);
      }
    },
  },
  created() {
    this.fetchTasks();
  },
};
</script>
<style>
#app {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.add-task-section {
  margin-bottom: 20px;
}

.add-task-input {
  width: calc(100% - 110px);
  padding: 10px;
  margin-right: 10px;
  border: 2px solid #2196f3;
  border-radius: 5px;
  font-size: 16px;
}

.add-task-btn {
  padding: 10px 20px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.add-task-btn:hover {
  background-color: #0d47a1;
}

.task-section {
  margin-bottom: 30px;
}

h1 {
  text-align: center;
  color: #009688;
}

h2 {
  color: #009688;
}

ul {
  list-style-type: none;
  padding: 0;
}

.task {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 5px;
  margin-bottom: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.task:hover {
  background-color: #e0e0e0;
  transform: translateY(-2px);
}

.task-info {
  flex-grow: 1;
}

.task-description {
  font-weight: bold;
  color: #333;
}

.task-date {
  font-size: 12px;
  color: #666;
}

.complete-btn,
.delete-btn {
  padding: 5px 10px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.complete-btn:hover,
.delete-btn:hover {
  background-color: #388e3c;
}

.no-task {
  text-align: center;
  font-style: italic;
  color: #888;
}
</style>
