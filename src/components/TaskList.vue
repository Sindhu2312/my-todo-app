<template>
  <div class="todo-app">
    <!-- Animated Background Elements -->
    <div class="background-animations">
      <!-- Animated Gradient Background -->
      <div class="gradient-bg"></div>
      
      <!-- Floating Geometric Shapes -->
      <div class="floating-shape circle shape-1"></div>
      <div class="floating-shape square shape-2"></div>
      <div class="floating-shape triangle shape-3"></div>
      <div class="floating-shape circle shape-4"></div>
      <div class="floating-shape square shape-5"></div>
      <div class="floating-shape triangle shape-6"></div>
      <div class="floating-shape circle shape-7"></div>
      <div class="floating-shape square shape-8"></div>
      
      <!-- Moving Task-related Particles -->
      <div class="moving-particle particle-1">✓</div>
      <div class="moving-particle particle-2">📝</div>
      <div class="moving-particle particle-3">⭐</div>
      <div class="moving-particle particle-4">🎯</div>
      <div class="moving-particle particle-5">✨</div>
      <div class="moving-particle particle-6">📋</div>
      <div class="moving-particle particle-7">🚀</div>
      <div class="moving-particle particle-8">💫</div>
      
      <!-- Animated Wave Lines -->
      <svg class="wave-animation" viewBox="0 0 1200 600" preserveAspectRatio="none">
        <path class="wave-path wave-1" d="M0,300 Q300,200 600,300 T1200,300 V600 H0 Z"></path>
        <path class="wave-path wave-2" d="M0,350 Q300,250 600,350 T1200,350 V600 H0 Z"></path>
        <path class="wave-path wave-3" d="M0,400 Q300,300 600,400 T1200,400 V600 H0 Z"></path>
      </svg>
      
      <!-- Orbiting Elements -->
      <div class="orbit-container orbit-1">
        <div class="orbit-element">🌟</div>
      </div>
      <div class="orbit-container orbit-2">
        <div class="orbit-element">💼</div>
      </div>
      <div class="orbit-container orbit-3">
        <div class="orbit-element">⚡</div>
      </div>
    </div>

    <div class="header">
      <h1>
        <!-- <span class="icon">📝</span> -->
        <span class="typewriter">📝 Small Steps. Big Wins.</span>
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
        <div class="datetime-container">
          <input
            v-model="newTaskDueDate"
            type="date"
            class="date-input"
            :min="minDate"
            placeholder="Due date"
          />
          <input
            v-model="newTaskDueTime"
            type="time"
            class="time-input"
            placeholder="Due time"
          />
        </div>
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
          :class="['task-item', `priority-${task.priority}`, { completed: task.done, expired: isTaskExpired(task) && !task.done }]"
        >
          <div class="task-content">
            <input 
              type="checkbox" 
              v-model="task.done" 
              class="task-checkbox"
              @change="updateTask(task)"
            />
            <div class="task-text-container" @dblclick="startEdit(task)">
              <div v-if="editingTask === task.id" class="edit-container">
                <input
                  v-model="editText"
                  @keyup.enter="saveEdit(task)"
                  @keyup.esc="cancelEdit"
                  @blur="saveEdit(task)"
                  class="edit-input"
                  ref="editInput"
                  placeholder="Edit task description..."
                />
                <div class="edit-datetime-section">
                  <label class="edit-datetime-label">Due Date & Time:</label>
                  <div class="edit-datetime-container">
                    <input
                      v-model="editDueDate"
                      type="date"
                      class="edit-date-input"
                      :min="minDate"
                      placeholder="Due date"
                      title="Edit due date"
                      @blur="saveEdit(task)"
                    />
                    <input
                      v-model="editDueTime"
                      type="time"
                      class="edit-time-input"
                      placeholder="Due time"
                      title="Edit due time"
                      @blur="saveEdit(task)"
                    />
                  </div>
                </div>
                <!-- edit-actions removed; inputs now auto-save on blur/enter -->
              </div>
              <span v-else :class="['task-text', { done: task.done, expired: isTaskExpired(task) && !task.done }]">
                {{ task.text }}
                <span v-if="isTaskExpired(task) && !task.done" class="expired-inline">
                  ⚠️ Expired {{ formatExpiredDate(task.dueDateTime) }}
                </span>
                <span v-if="task.dueDateTime" class="due-date-display">
                  📅 {{ formatDueDate(task.dueDateTime) }}
                </span>
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
      editDueDate: '',
      editDueTime: '',
      showError: false,
      nextId: 1,
      newTaskDueDate: '',
      newTaskDueTime: '',
      minDate: new Date().toISOString().split('T')[0],
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
  mounted() {
    this.checkExpiredTasks();
    this.expiredInterval = setInterval(this.checkExpiredTasks, 60000); // check every minute
  },
  beforeUnmount() {
    clearInterval(this.expiredInterval);
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

      // Combine date and time for dueDateTime
      let dueDateTime = null;
      if (this.newTaskDueDate) {
        dueDateTime = this.newTaskDueDate;
        if (this.newTaskDueTime) {
          dueDateTime += 'T' + this.newTaskDueTime;
        } else {
          dueDateTime += 'T23:59'; // default to end of day
        }
      }

      const newTask = {
        id: this.nextId++,
        text,
        done: false,
        priority: this.newTaskPriority,
        createdAt: new Date().toISOString(),
        dueDateTime
      };

      this.tasks.unshift(newTask); // Add to beginning for better UX
      this.newTask = '';
      this.newTaskPriority = 'medium';
      this.newTaskDueDate = '';
      this.newTaskDueTime = '';
      this.showError = false;
    },

    removeTask(index) {
      if (confirm('Are you sure you want to delete this task?')) {
        this.tasks.splice(index, 1);
      }
    },

    startEdit(task) {
      if (task.done) return; // Don't edit completed tasks
      // Toggle edit mode: if already editing this task, close it
      if (this.editingTask === task.id) {
        this.cancelEdit();
        return;
      }
      // Only allow one edit at a time
      if (this.editingTask !== null && this.editingTask !== task.id) {
        this.cancelEdit();
      }
      this.editingTask = task.id;
      this.editText = task.text;
      // Parse existing date/time if available
      if (task.dueDateTime) {
        try {
          const dateTime = new Date(task.dueDateTime);
          this.editDueDate = dateTime.toISOString().split('T')[0];
          const hours = dateTime.getHours().toString().padStart(2, '0');
          const minutes = dateTime.getMinutes().toString().padStart(2, '0');
          this.editDueTime = `${hours}:${minutes}`;
        } catch (error) {
          console.error('Error parsing date:', error);
          this.editDueDate = '';
          this.editDueTime = '';
        }
      } else {
        this.editDueDate = '';
        this.editDueTime = '';
      }
      this.$nextTick(() => {
        const editInput = this.$refs.editInput;
        if (editInput && editInput.focus) {
          editInput.focus();
          editInput.select && editInput.select();
        } else if (editInput && editInput.length) {
          editInput[0].focus();
          editInput[0].select && editInput[0].select();
        }
      });
    },

    saveEdit(task) {
      const text = this.editText.trim();
      if (text) {
        task.text = text;
      }
      // Update due date/time
      let dueDateTime = null;
      if (this.editDueDate) {
        dueDateTime = this.editDueDate;
        if (this.editDueTime) {
          dueDateTime += 'T' + this.editDueTime;
        } else {
          dueDateTime += 'T23:59'; // default to end of day
        }
      }
      task.dueDateTime = dueDateTime;
      // Force reactivity update
      const taskIndex = this.tasks.findIndex(t => t.id === task.id);
      if (taskIndex !== -1) {
        this.$set(this.tasks, taskIndex, { ...task });
      }
      // Once saved, cancel edit to reset state
      this.cancelEdit();
      // Blur date/time pickers to close dropdowns
      this.$nextTick(() => {
        const dateInput = document.querySelector('.edit-date-input');
        const timeInput = document.querySelector('.edit-time-input');
        if (dateInput) dateInput.blur();
        if (timeInput) timeInput.blur();
      });
    },

    cancelEdit() {
      this.editingTask = null;
      this.editText = '';
      this.editDueDate = '';
      this.editDueTime = '';
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
          createdAt: task.createdAt || new Date().toISOString(),
          dueDateTime: task.dueDateTime || null
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
    },

    isTaskExpired(task) {
      if (!task.dueDateTime) return false;
      return !task.done && new Date(task.dueDateTime) < new Date();
    },
    checkExpiredTasks() {
      // Just trigger reactivity to update visual indicators
      // No intrusive alerts - users can see expired tasks with visual warnings
      this.tasks = [...this.tasks];
    },

    formatDueDate(dueDateTime) {
      if (!dueDateTime) return '';
      const date = new Date(dueDateTime);
      const now = new Date();
      
      // Check if it's today
      if (date.toDateString() === now.toDateString()) {
        return `Today at ${date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}`;
      }
      
      // Check if it's tomorrow
      const tomorrow = new Date(now);
      tomorrow.setDate(tomorrow.getDate() + 1);
      if (date.toDateString() === tomorrow.toDateString()) {
        return `Tomorrow at ${date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}`;
      }
      
      // Check if it's yesterday
      const yesterday = new Date(now);
      yesterday.setDate(yesterday.getDate() - 1);
      if (date.toDateString() === yesterday.toDateString()) {
        return `Yesterday at ${date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}`;
      }
      
      return date.toLocaleDateString() + ' at ' + date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    },

    formatExpiredDate(dueDateTime) {
      if (!dueDateTime) return '';
      const date = new Date(dueDateTime);
      const now = new Date();
      const diffTime = now - date;
      const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));
      const diffHours = Math.floor(diffTime / (1000 * 60 * 60));
      const diffMinutes = Math.floor(diffTime / (1000 * 60));
      
      if (diffDays > 0) {
        return `${diffDays} day${diffDays > 1 ? 's' : ''} ago`;
      } else if (diffHours > 0) {
        return `${diffHours} hour${diffHours > 1 ? 's' : ''} ago`;
      } else if (diffMinutes > 0) {
        return `${diffMinutes} minute${diffMinutes > 1 ? 's' : ''} ago`;
      } else {
        return 'Just now';
      }
    },
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

.datetime-container {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  background: #f8fafc;
  padding: 0.5rem;
  border-radius: 8px;
  border: 2px solid #e2e8f0;
  transition: all 0.2s;
}

.datetime-container:focus-within {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.date-input, .time-input {
  border: none;
  background: transparent;
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
  color: #4a5568;
  border-radius: 4px;
  transition: all 0.2s;
}

.date-input:focus, .time-input:focus {
  outline: none;
  background: white;
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.2);
}

.date-input {
  min-width: 130px;
}

.time-input {
  min-width: 80px;
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

.task-item.expired {
  background: #fef2f2;
  border-left: 4px solid #f56565;
}

.task-item.expired:hover {
  background: #fed7d7;
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

.task-text.expired {
  color: #e53e3e;
  font-weight: 500;
}

.due-date-display {
  display: inline-block;
  margin-left: 0.75rem;
  font-size: 0.75rem;
  color: #718096;
  background: #edf2f7;
  padding: 0.2rem 0.5rem;
  border-radius: 12px;
  font-weight: 500;
}

.expired-inline {
  display: inline-block;
  margin-left: 0.5rem;
  font-size: 0.75rem;
  color: #c53030;
  background: #fed7d7;
  padding: 0.2rem 0.5rem;
  border-radius: 12px;
  font-weight: 600;
  animation: pulse 2s ease-in-out infinite;
}

.edit-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  padding: 0.5rem;
  background: #f8fafc;
  border-radius: 8px;
  border: 2px solid #e2e8f0;
}

.edit-datetime-section {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.edit-datetime-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #4a5568;
  margin-bottom: 0.25rem;
}

.edit-datetime-container {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  background: #f8fafc;
  padding: 0.5rem;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.edit-date-input, .edit-time-input {
  border: none;
  background: transparent;
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
  color: #4a5568;
  border-radius: 4px;
  transition: all 0.2s;
}

.edit-date-input:focus, .edit-time-input:focus {
  outline: none;
  background: white;
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.2);
}

.edit-date-input {
  min-width: 130px;
}

.edit-time-input {
  min-width: 80px;
}

.edit-actions {
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
  margin-top: 0.5rem;
}

.save-btn, .cancel-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.save-btn {
  background: #38a169;
  color: white;
}

.save-btn:hover {
  background: #2f855a;
  transform: translateY(-1px);
}

.cancel-btn {
  background: #e2e8f0;
  color: #4a5568;
}

.cancel-btn:hover {
  background: #cbd5e0;
}

.expired-badge {
  display: inline-block;
  margin-left: 0.5rem;
  font-size: 0.65rem;
  padding: 0.2rem 0.4rem;
  background: #fed7d7;
  color: #c53030;
  border-radius: 8px;
  font-weight: 600;
  text-transform: uppercase;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { 
    opacity: 1; 
    transform: scale(1);
  }
  50% { 
    opacity: 0.8; 
    transform: scale(1.05);
  }
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

  .datetime-container {
    width: 100%;
    flex-direction: column;
    gap: 0.75rem;
  }

  .date-input, .time-input {
    width: 100%;
    min-width: unset;
    padding: 0.5rem;
    font-size: 1rem;
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

  .edit-datetime-container {
    flex-direction: column;
    gap: 0.5rem;
  }

  .edit-date-input, .edit-time-input {
    width: 100%;
    min-width: unset;
    padding: 0.5rem;
    font-size: 1rem;
  }

  .expired-inline {
    display: block;
    margin-left: 0;
    margin-top: 0.25rem;
    font-size: 0.75rem;
  }

  .due-date-display {
    display: block;
    margin-left: 0;
    margin-top: 0.25rem;
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

/* Animated Background Elements */
.background-animations {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: -1;
  overflow: hidden;
}

/* Animated Gradient Background */
.gradient-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 120%;
  height: 120%;
  background: linear-gradient(45deg, #667eea, #764ba2, #f093fb, #f5576c, #4facfe, #00f2fe);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
  transform: translate(-10%, -10%);
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* Floating Geometric Shapes */
.floating-shape {
  position: absolute;
  opacity: 0.15;
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
  animation-direction: alternate;
}

.circle {
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.8);
}

.square {
  background: rgba(255, 255, 255, 0.6);
  border-radius: 10px;
}

.triangle {
  width: 0;
  height: 0;
  background: transparent;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-bottom: 35px solid rgba(255, 255, 255, 0.7);
}

.shape-1 {
  width: 80px;
  height: 80px;
  top: 10%;
  left: 10%;
  animation: float1 8s infinite;
}

.shape-2 {
  width: 60px;
  height: 60px;
  top: 20%;
  right: 15%;
  animation: float2 10s infinite;
}

.shape-3 {
  top: 40%;
  left: 5%;
  animation: float3 12s infinite;
}

.shape-4 {
  width: 100px;
  height: 100px;
  top: 60%;
  right: 10%;
  animation: float4 9s infinite;
}

.shape-5 {
  width: 40px;
  height: 40px;
  top: 80%;
  left: 20%;
  animation: float5 11s infinite;
}

.shape-6 {
  top: 30%;
  right: 5%;
  animation: float6 7s infinite;
}

.shape-7 {
  width: 120px;
  height: 120px;
  top: 70%;
  right: 25%;
  animation: float7 13s infinite;
}

.shape-8 {
  width: 50px;
  height: 50px;
  top: 15%;
  left: 40%;
  animation: float8 6s infinite;
}

/* Floating Animation Keyframes */
@keyframes float1 {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  25% { transform: translate(30px, -20px) rotate(90deg); }
  50% { transform: translate(-20px, -40px) rotate(180deg); }
  75% { transform: translate(-40px, 20px) rotate(270deg); }
}

@keyframes float2 {
  0%, 100% { transform: translate(0, 0) rotate(0deg) scale(1); }
  33% { transform: translate(-40px, 30px) rotate(120deg) scale(1.2); }
  66% { transform: translate(25px, -35px) rotate(240deg) scale(0.8); }
}

@keyframes float3 {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  50% { transform: translate(60px, -50px) rotate(180deg); }
}

@keyframes float4 {
  0%, 100% { transform: translate(0, 0) scale(1); }
  25% { transform: translate(-30px, -25px) scale(1.1); }
  50% { transform: translate(20px, -50px) scale(0.9); }
  75% { transform: translate(40px, 10px) scale(1.05); }
}

@keyframes float5 {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  20% { transform: translate(25px, -30px) rotate(72deg); }
  40% { transform: translate(-15px, -45px) rotate(144deg); }
  60% { transform: translate(-35px, -10px) rotate(216deg); }
  80% { transform: translate(10px, 25px) rotate(288deg); }
}

@keyframes float6 {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  50% { transform: translate(-45px, 40px) rotate(360deg); }
}

@keyframes float7 {
  0%, 100% { transform: translate(0, 0) scale(1) rotate(0deg); }
  33% { transform: translate(20px, -30px) scale(0.8) rotate(120deg); }
  66% { transform: translate(-25px, 35px) scale(1.1) rotate(240deg); }
}

@keyframes float8 {
  0%, 100% { transform: translate(0, 0); }
  25% { transform: translate(35px, 20px); }
  50% { transform: translate(-20px, 40px); }
  75% { transform: translate(-40px, -15px); }
}

/* Moving Particles */
.moving-particle {
  position: absolute;
  font-size: 24px;
  opacity: 0.6;
  pointer-events: none;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}

.particle-1 {
  top: 5%;
  left: -50px;
  animation: moveAcross1 20s infinite;
  color: #4ade80;
}

.particle-2 {
  top: 25%;
  right: -50px;
  animation: moveAcross2 25s infinite;
  color: #60a5fa;
}

.particle-3 {
  top: 45%;
  left: -50px;
  animation: moveAcross3 18s infinite;
  color: #fbbf24;
}

.particle-4 {
  top: 65%;
  right: -50px;
  animation: moveAcross4 22s infinite;
  color: #f87171;
}

.particle-5 {
  top: 85%;
  left: -50px;
  animation: moveAcross5 16s infinite;
  color: #a78bfa;
}

.particle-6 {
  top: 35%;
  left: -50px;
  animation: moveAcross6 28s infinite;
  color: #34d399;
}

.particle-7 {
  top: 55%;
  right: -50px;
  animation: moveAcross7 24s infinite;
  color: #fb7185;
}

.particle-8 {
  top: 75%;
  left: -50px;
  animation: moveAcross8 19s infinite;
  color: #fcd34d;
}

@keyframes moveAcross1 {
  0% { transform: translateX(0) translateY(0) rotate(0deg); }
  25% { transform: translateX(25vw) translateY(-20px) rotate(90deg); }
  50% { transform: translateX(50vw) translateY(30px) rotate(180deg); }
  75% { transform: translateX(75vw) translateY(-10px) rotate(270deg); }
  100% { transform: translateX(calc(100vw + 50px)) translateY(20px) rotate(360deg); }
}

@keyframes moveAcross2 {
  0% { transform: translateX(0) translateY(0) scale(1); }
  30% { transform: translateX(-30vw) translateY(25px) scale(1.2); }
  60% { transform: translateX(-60vw) translateY(-15px) scale(0.8); }
  100% { transform: translateX(calc(-100vw - 50px)) translateY(35px) scale(1); }
}

@keyframes moveAcross3 {
  0% { transform: translateX(0) translateY(0) rotate(0deg); }
  100% { transform: translateX(calc(100vw + 50px)) translateY(-40px) rotate(720deg); }
}

@keyframes moveAcross4 {
  0% { transform: translateX(0) translateY(0) scale(1); }
  50% { transform: translateX(-50vw) translateY(-30px) scale(1.5); }
  100% { transform: translateX(calc(-100vw - 50px)) translateY(20px) scale(1); }
}

@keyframes moveAcross5 {
  0% { transform: translateX(0) translateY(0); }
  20% { transform: translateX(20vw) translateY(-25px); }
  40% { transform: translateX(40vw) translateY(15px); }
  60% { transform: translateX(60vw) translateY(-35px); }
  80% { transform: translateX(80vw) translateY(25px); }
  100% { transform: translateX(calc(100vw + 50px)) translateY(-15px); }
}

@keyframes moveAcross6 {
  0% { transform: translateX(0) translateY(0) rotate(0deg) scale(1); }
  100% { transform: translateX(calc(100vw + 50px)) translateY(45px) rotate(-360deg) scale(1.3); }
}

@keyframes moveAcross7 {
  0% { transform: translateX(0) translateY(0); }
  25% { transform: translateX(-25vw) translateY(-20px); }
  50% { transform: translateX(-50vw) translateY(40px); }
  75% { transform: translateX(-75vw) translateY(-30px); }
  100% { transform: translateX(calc(-100vw - 50px)) translateY(15px); }
}

@keyframes moveAcross8 {
  0% { transform: translateX(0) translateY(0) rotate(0deg); }
  100% { transform: translateX(calc(100vw + 50px)) translateY(-25px) rotate(540deg); }
}

/* Wave Animation */
.wave-animation {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0.1;
}

.wave-path {
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
  animation-direction: alternate;
}

.wave-1 {
  fill: rgba(255, 255, 255, 0.3);
  animation: waveMove1 8s infinite;
}

.wave-2 {
  fill: rgba(255, 255, 255, 0.2);
  animation: waveMove2 12s infinite;
}

.wave-3 {
  fill: rgba(255, 255, 255, 0.1);
  animation: waveMove3 10s infinite;
}

@keyframes waveMove1 {
  0%, 100% { d: path("M0,300 Q300,200 600,300 T1200,300 V600 H0 Z"); }
  50% { d: path("M0,250 Q300,350 600,250 T1200,250 V600 H0 Z"); }
}

@keyframes waveMove2 {
  0%, 100% { d: path("M0,350 Q300,250 600,350 T1200,350 V600 H0 Z"); }
  50% { d: path("M0,400 Q300,300 600,400 T1200,400 V600 H0 Z"); }
}

@keyframes waveMove3 {
  0%, 100% { d: path("M0,400 Q300,300 600,400 T1200,400 V600 H0 Z"); }
  50% { d: path("M0,450 Q300,350 600,450 T1200,450 V600 H0 Z"); }
}

/* Orbiting Elements */
.orbit-container {
  position: absolute;
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 50%;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}

.orbit-1 {
  width: 200px;
  height: 200px;
  top: 20%;
  left: 10%;
  animation: rotate 20s infinite;
}

.orbit-2 {
  width: 300px;
  height: 300px;
  top: 50%;
  right: 5%;
  animation: rotate 30s infinite reverse;
}

.orbit-3 {
  width: 150px;
  height: 150px;
  bottom: 20%;
  left: 30%;
  animation: rotate 15s infinite;
}

.orbit-element {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 20px;
  opacity: 0.7;
  animation: counterRotate 20s linear infinite;
}

.orbit-2 .orbit-element {
  animation: counterRotate 30s linear infinite reverse;
}

.orbit-3 .orbit-element {
  animation: counterRotate 15s linear infinite;
}

@keyframes rotate {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@keyframes counterRotate {
  0% { transform: translateX(-50%) rotate(0deg); }
  100% { transform: translateX(-50%) rotate(-360deg); }
}

/* Make sure todo-app has proper positioning */
.todo-app {
  position: relative;
  z-index: 1;
}
</style>