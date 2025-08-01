<template>
  <div class="about">
    <div class="about-content">
      <h2>About To-Do App</h2>
      <p class="description">
        A modern, feature-rich todo application built with Vue.js 3 and Vite. 
        This app demonstrates best practices in Vue development with a clean, 
        responsive design and intuitive user experience.
      </p>
      
      <div class="features">
        <h3>Features</h3>
        <ul>
          <li>✅ Add, edit, and delete tasks</li>
          <li>📋 Task priority levels (High, Medium, Low)</li>
          <li>🔍 Filter tasks by status (All, Active, Completed)</li>
          <li>📊 Task statistics and counters</li>
          <li>💾 Persistent storage with localStorage</li>
          <li>📱 Responsive design for all devices</li>
          <li>🎨 Modern UI with smooth animations</li>
          <li>⚡ Fast and lightweight</li>
        </ul>
      </div>

      <div class="tech-stack">
        <h3>Technology Stack</h3>
        <div class="tech-items">
          <span class="tech-item">Vue.js 3</span>
          <span class="tech-item">Vue Router 4</span>
          <span class="tech-item">Vite</span>
          <span class="tech-item">CSS3</span>
          <span class="tech-item">LocalStorage API</span>
        </div>
      </div>

      <div class="tips">
        <h3>Usage Tips</h3>
        <ul>
          <li>💡 Double-click any task to edit it quickly</li>
          <li>🎯 Use priority levels to organize important tasks</li>
          <li>⌨️ Press Enter to add tasks quickly</li>
          <li>🗂️ Use filters to focus on specific task types</li>
          <li>🧹 Use bulk actions to manage multiple tasks at once</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'About'
};
</script>

<style scoped>
.about {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.about-content {
  background: white;
  padding: 3rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

h2 {
  color: #2d3748;
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-align: center;
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

h3 {
  color: #4a5568;
  font-size: 1.5rem;
  margin: 2rem 0 1rem 0;
  border-bottom: 2px solid #e2e8f0;
  padding-bottom: 0.5rem;
}

.description {
  font-size: 1.125rem;
  color: #718096;
  line-height: 1.6;
  text-align: center;
  margin-bottom: 2rem;
}

.features ul,
.tips ul {
  list-style: none;
  padding: 0;
}

.features li,
.tips li {
  padding: 0.75rem 0;
  border-bottom: 1px solid #f1f5f9;
  color: #4a5568;
  font-size: 1rem;
  display: flex;
  align-items: center;
}

.features li:last-child,
.tips li:last-child {
  border-bottom: none;
}

.tech-items {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin-top: 1rem;
}

.tech-item {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.875rem;
  font-weight: 600;
}

@media (max-width: 640px) {
  .about {
    padding: 1rem;
  }
  
  .about-content {
    padding: 2rem 1.5rem;
  }
  
  h2 {
    font-size: 2rem;
  }
  
  .tech-items {
    justify-content: center;
  }
}
</style>
