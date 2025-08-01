<template>
  <div class="todo-app">
    <div class="header">
      <h1>
        <span class="icon">📝</span>
        <span class="typewriter">Small Steps. Big Wins.</span>
      </h1>
      <div class="stats">
        <span class="stat">Total: {{ tasks.length }}</span>
        <span class="stat">Active: {{ activeTasks.length }}</span>
        <span class="stat">Completed: {{ completedTasks.length }}</span>
      </div>
    </div>

    <div class="add-task-section">
      <form @submit.prevent="addTask" class="add-task-form">
        <input
          v-model="newTask"
          placeholder="Add a new task..."
          class="task-input"
          :class="{ 'error': showError }"
          ref="taskInput"
        />
        <select v-model="newTaskPriority" class="priority-select">
          <option value="low">Low</option>
          <option value="medium">Medium</option>
          <option value="high">High</option>
        </select>
        <button type="submit" class="add-btn" :disabled="!newTask.trim()">
          Add Task
        </button>
      </form>
      <div v-if="showError" class="error-message">
        Please enter a task description
      </div>
    </div>

    <div class="filters" v-if="tasks.length > 0">
      <button 
        v-for="filter in filters" 
        :key="filter.value"
        @click="currentFilter = filter.value"
        :class="['filter-btn', { active: currentFilter === filter.value }]"
      >
        {{ filter.label }} ({{ getFilterCount(filter.value) }})
      </button>
    </div>

    <div class="tasks-container" v-if="filteredTasks.length > 0">
      <transition-group name="task" tag="ul" class="task-list">
        <li 
          v-for="task in filteredTasks" 
          :key="task.id"
          :class="['task-item', `priority-${task.priority}`, { completed: task.done }]"
        >
          <div class="task-content">
            <input 
              type="checkbox" 
              v-model="task.done" 
              class="task-checkbox"
              @change="updateTask(task)"
            />
            <div class="task-text-container" @dblclick="startEdit(task)">
              <input
                v-if="editingTask === task.id"
                v-model="editText"
                @blur="saveEdit(task)"
                @keyup.enter="saveEdit(task)"
                @keyup.esc="cancelEdit"
                class="edit-input"
                ref="editInput"
              />
              <span v-else :class="['task-text', { done: task.done }]">
                {{ task.text }}
              </span>
              <span :class="['priority-badge', `priority-${task.priority}`]">
                {{ task.priority }}
              </span>
            </div>
          </div>
          <div class="task-actions">
            <button 
              @click="startEdit(task)" 
              class="edit-btn"
              :disabled="task.done"
              title="Edit task"
            >
              ✏️
            </button>
            <button 
              @click="removeTask(getOriginalIndex(task.id))" 
              class="delete-btn"
              title="Delete task"
            >
              🗑️
            </button>
          </div>
        </li>
      </transition-group>
    </div>

    <div v-else-if="tasks.length > 0" class="no-tasks">
      <p>No {{ currentFilter }} tasks found</p>
    </div>

    <div v-else class="empty-state">
      <p>🎉 No tasks yet! Add one above to get started.</p>
    </div>

    <div class="bulk-actions" v-if="tasks.length > 0">
      <button 
        @click="markAllComplete" 
        class="bulk-btn mark-all"
        :disabled="activeTasks.length === 0"
      >
        ✔️ Mark All Complete
      </button>
      <button 
        @click="clearCompleted" 
        class="bulk-btn clear-all"
        :disabled="completedTasks.length === 0"
      >
        🧹 Clear Completed ({{ completedTasks.length }})
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newTask: '',
      newTaskPriority: 'medium',
      tasks: [],
      currentFilter: 'all',
      editingTask: null,
      editText: '',
      showError: false,
      nextId: 1,
      filters: [
        { label: 'All', value: 'all' },
        { label: 'Active', value: 'active' },
        { label: 'Completed', value: 'completed' }
      ]
    };
  },
  computed: {
    filteredTasks() {
      switch (this.currentFilter) {
        case 'active':
          return this.tasks.filter(task => !task.done);
        case 'completed':
          return this.tasks.filter(task => task.done);
        default:
          return this.tasks;
      }
    },
    activeTasks() {
      return this.tasks.filter(task => !task.done);
    },
    completedTasks() {
      return this.tasks.filter(task => task.done);
    }
  },
  created() {
    this.loadTasks();
  },
  watch: {
    tasks: {
      handler() {
        this.saveTasks();
      },
      deep: true
    }
  },
  methods: {
    addTask() {
      const text = this.newTask.trim();
      if (!text) {
        this.showError = true;
        this.$refs.taskInput.focus();
        setTimeout(() => {
          this.showError = false;
        }, 3000);
        return;
      }

      const newTask = {
        id: this.nextId++,
        text,
        done: false,
        priority: this.newTaskPriority,
        createdAt: new Date().toISOString()
      };

      this.tasks.unshift(newTask); // Add to beginning for better UX
      this.newTask = '';
      this.newTaskPriority = 'medium';
      this.showError = false;
    },

    removeTask(index) {
      if (confirm('Are you sure you want to delete this task?')) {
        this.tasks.splice(index, 1);
      }
    },

    startEdit(task) {
      if (task.done) return; // Don't edit completed tasks
      this.editingTask = task.id;
      this.editText = task.text;
      this.$nextTick(() => {
        const editInput = this.$refs.editInput;
        if (editInput && editInput.length) {
          editInput[0].focus();
          editInput[0].select();
        }
      });
    },

    saveEdit(task) {
      const text = this.editText.trim();
      if (text && text !== task.text) {
        task.text = text;
      }
      this.cancelEdit();
    },

    cancelEdit() {
      this.editingTask = null;
      this.editText = '';
    },

    updateTask(task) {
      // Trigger reactivity for localStorage save
      this.tasks = [...this.tasks];
    },

    markAllComplete() {
      this.tasks.forEach(task => {
        task.done = true;
      });
    },

    clearCompleted() {
      if (confirm(`Delete ${this.completedTasks.length} completed tasks?`)) {
        this.tasks = this.tasks.filter(task => !task.done);
      }
    },

    getFilterCount(filter) {
      switch (filter) {
        case 'active':
          return this.activeTasks.length;
        case 'completed':
          return this.completedTasks.length;
        default:
          return this.tasks.length;
      }
    },

    getOriginalIndex(taskId) {
      return this.tasks.findIndex(task => task.id === taskId);
    },

    loadTasks() {
      try {
        const savedTasks = localStorage.getItem('vue-todo-tasks');
        const savedNextId = localStorage.getItem('vue-todo-nextId');
        
        if (savedTasks) {
          this.tasks = JSON.parse(savedTasks);
        }
        
        if (savedNextId) {
          this.nextId = parseInt(savedNextId);
        } else if (this.tasks.length > 0) {
          // Generate next ID based on existing tasks
          this.nextId = Math.max(...this.tasks.map(t => t.id || 0)) + 1;
        }

        // Ensure all tasks have required properties
        this.tasks = this.tasks.map(task => ({
          id: task.id || this.nextId++,
          text: task.text || '',
          done: task.done || false,
          priority: task.priority || 'medium',
          createdAt: task.createdAt || new Date().toISOString()
        }));
      } catch (error) {
        console.error('Error loading tasks:', error);
        this.tasks = [];
        this.nextId = 1;
      }
    },

    saveTasks() {
      try {
        localStorage.setItem('vue-todo-tasks', JSON.stringify(this.tasks));
        localStorage.setItem('vue-todo-nextId', this.nextId.toString());
      } catch (error) {
        console.error('Error saving tasks:', error);
      }
    }
  }
};
</script>

<style scoped>
.todo-app {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #f8fafc;
  /* Removed fixed viewport height to prevent initial scroll */
  /* min-height: 100vh; */
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

 .header h1 {
  color: #2d3748;
  margin-bottom: 1rem;
  font-size: 2.5rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.header h1 .icon {
  font-size: 1.8rem;
  margin-right: 0.5rem;
  vertical-align: middle;
}

.typewriter {
  overflow: hidden;
  border-right: .15em solid #667eea;
  white-space: nowrap;
  display: inline-block;
  animation: typing 3s steps(30, end), blink-caret .75s step-end infinite;
}

 @keyframes typing {
  from { width: 0; }
  to { width: 22ch; }
}

@keyframes blink-caret {
  from, to { border-color: transparent; }
  50% { border-color: #667eea; }
}

.stats {
  display: flex;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.stat {
  background: #e2e8f0;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.875rem;
  font-weight: 600;
  color: #4a5568;
}

.add-task-section {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.add-task-form {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  flex-wrap: wrap;
}

.task-input {
  flex: 1;
  min-width: 200px;
  padding: 0.75rem 1rem;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
  transition: all 0.2s;
}

.task-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.task-input.error {
  border-color: #e53e3e;
  box-shadow: 0 0 0 3px rgba(229, 62, 62, 0.1);
}

.priority-select {
  /* Adjusted padding for custom arrow and ensure text visibility */
  padding: 0.75rem 2rem 0.75rem 0.75rem;
  min-width: 100px;  /* Ensure select box is wide enough */
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
  background: white;
  cursor: pointer;
  /* Hide default arrow */
  appearance: none;
  -webkit-appearance: none;
  /* Custom dropdown arrow */
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 10 6'%3E%3Cpath d='M0 0 L5 6 L10 0 Z' fill='%23667676'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 0.75rem center;
  background-size: 0.65em;
}

.add-btn {
  background: #667eea;
  color: white;
  border: none;
  padding: 0.75rem 2rem;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.add-btn:hover:not(:disabled) {
  background: #5a67d8;
  transform: translateY(-1px);
}

.add-btn:disabled {
  background: #a0aec0;
  cursor: not-allowed;
}

.error-message {
  color: #e53e3e;
  font-size: 0.875rem;
  margin-top: 0.5rem;
  font-weight: 500;
}

.filters {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
}

.filter-btn {
  padding: 0.5rem 1rem;
  border: 2px solid #e2e8f0;
  background: white;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.875rem;
  font-weight: 600;
  transition: all 0.2s;
  color: #4a5568;
}

.filter-btn:hover {
  border-color: #cbd5e0;
}

.filter-btn.active {
  background: #667eea;
  border-color: #667eea;
  color: white;
}

.tasks-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.task-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.5rem;
  border-bottom: 1px solid #f1f5f9;
  transition: all 0.3s;
  position: relative;
}

.task-item:last-child {
  border-bottom: none;
}

.task-item:hover {
  background: #f8fafc;
}

.task-item.completed {
  opacity: 0.7;
}

.priority-high {
  border-left: 4px solid #e53e3e;
}

.priority-medium {
  border-left: 4px solid #ed8936;
}

.priority-low {
  border-left: 4px solid #38a169;
}

.task-content {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex: 1;
}

.task-checkbox {
  width: 1.25rem;
  height: 1.25rem;
  cursor: pointer;
}

.task-text-container {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.task-text {
  font-size: 1rem;
  color: #2d3748;
  transition: all 0.2s;
  word-break: break-word;
}

.task-text.done {
  text-decoration: line-through;
  color: #a0aec0;
}

.edit-input {
  flex: 1;
  padding: 0.5rem;
  border: 2px solid #667eea;
  border-radius: 4px;
  font-size: 1rem;
  background: white;
}

.priority-badge {
  font-size: 0.75rem;
  padding: 0.25rem 0.5rem;
  border-radius: 12px;
  font-weight: 600;
  text-transform: uppercase;
}

.priority-badge.priority-high {
  background: #fed7d7;
  color: #c53030;
}

.priority-badge.priority-medium {
  background: #feebc8;
  color: #c05621;
}

.priority-badge.priority-low {
  background: #c6f6d5;
  color: #2f855a;
}

.task-actions {
  display: flex;
  gap: 0.5rem;
}

.edit-btn, .delete-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 6px;
  transition: all 0.2s;
  font-size: 1rem;
}

.edit-btn:hover:not(:disabled) {
  background: #edf2f7;
}

.edit-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.delete-btn:hover {
  background: #fed7d7;
}

.no-tasks, .empty-state {
  text-align: center;
  padding: 3rem 2rem;
  color: #718096;
  font-size: 1.125rem;
}

/* Designs for empty and no tasks states */
.no-tasks-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2rem;
  background: #edf2f7;
  border-radius: 12px;
  margin: 1rem 0;
}
.no-tasks-icon {
  font-size: 2rem;
  color: #667eea;
  margin-bottom: 1rem;
}
.clear-filter-btn {
  margin-top: 1rem;
  background: #667eea;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
}
.clear-filter-btn:hover {
  background: #5a67d8;
}

.empty-state-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2rem;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}
.empty-state-image {
  width: 120px;
  opacity: 0.6;
  margin-bottom: 1rem;
}
.empty-state-title {
  font-size: 1.5rem;
  color: #4a5568;
  margin-bottom: 0.5rem;
}
.empty-state-text {
  font-size: 1rem;
  color: #718096;
}

/* Animations */
.task-enter-active, .task-leave-active {
  transition: all 0.3s ease;
}

.task-enter-from {
  opacity: 0;
  transform: translateY(-20px);
}

.task-leave-to {
  opacity: 0;
  transform: translateX(100px);
}

/* Responsive design */
@media (max-width: 640px) {
  .todo-app {
    padding: 1rem;
  }
  
  .add-task-form {
    flex-direction: column;
  }
  
  .task-input {
    min-width: unset;
    width: 100%;
  }
  
  .priority-select, .add-btn {
    width: 100%;
  }
  
  .task-item {
    padding: 1rem;
    flex-direction: column;
    align-items: stretch;
    gap: 1rem;
  }
  
  .task-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.75rem;
  }
  
  .task-text-container {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
  
  .task-actions {
    justify-content: flex-end;
  }
  
  .bulk-actions {
    flex-direction: column;
  }
  
  .bulk-btn {
    width: 100%;
  }
}

/* Success styling for Mark All Complete */
.bulk-btn.mark-all {
  background: #38a169;
  color: white;
  border-color: #38a169;
}
.bulk-btn.mark-all:hover:not(:disabled) {
  background: #2f855a;
}

/* Danger styling for Clear Completed */
.bulk-btn.clear-all {
  background: #e53e3e;
  color: white;
  border-color: #e53e3e;
}
.bulk-btn.clear-all:hover:not(:disabled) {
  background: #c53030;
}

.bulk-actions {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-top: 3rem;         /* extra space above buttons */
    padding-top: 1rem;        /* additional padding */
    border-top: 1px solid #e2e8f0; /* subtle separator */
    flex-wrap: wrap;
  }

  .bulk-btn {
  padding: 0.75rem 2rem;      /* increased horizontal padding */
  }
</style>