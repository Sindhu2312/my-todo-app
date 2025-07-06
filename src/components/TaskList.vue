<template>
  <div class="app">
    <h1>📝 My Vue To-Do List</h1>

    <input
      v-model="newTask"
      @keyup.enter="addTask"
      placeholder="Add a task"
    />
    <button @click="addTask">Add</button>

    <ul>
      <li v-for="(task, index) in tasks" :key="index">
        <input type="checkbox" v-model="task.done" />
        <span :class="{ done: task.done }">{{ task.text }}</span>
        <button @click="removeTask(index)">❌</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newTask: '',
      tasks: []
    };
  },
  created() {
    const savedTasks = localStorage.getItem('tasks');
    if (savedTasks) {
      this.tasks = JSON.parse(savedTasks);
    }
  },
  watch: {
    tasks: {
      handler(newTasks) {
        localStorage.setItem('tasks', JSON.stringify(newTasks));
      },
      deep: true
    }
  },
  methods: {
    addTask() {
      const text = this.newTask.trim();
      if (text) {
        this.tasks.push({ text, done: false });
        this.newTask = '';
      }
    },
    removeTask(index) {
      this.tasks.splice(index, 1);
    }
  }
};
</script>

<style>
.app {
  max-width: 400px;
  margin: 2rem auto;
  font-family: sans-serif;
}

.done {
  text-decoration: line-through;
  color: gray;
}

input[type="text"] {
  padding: 5px;
  width: 70%;
  margin-right: 5px;
}

button {
  padding: 5px 10px;
}
</style>