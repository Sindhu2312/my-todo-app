<template>
  <div id="app">
    <nav class="navbar">
      <div class="nav-container">
        <router-link to="/home" class="nav-brand">📝 To-Do App</router-link>
        <div class="nav-links">
          <router-link to="/home" class="nav-link">Tasks</router-link>
          <router-link to="/about" class="nav-link">About</router-link>
        </div>
      </div>
    </nav>
    <main class="main-content">
      <router-view />
    </main>
  </div>
</template>

<script>
export default {};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.navbar {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

.nav-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.nav-brand {
  font-size: 1.5rem;
  font-weight: 700;
  color: #667eea;
  text-decoration: none;
  transition: color 0.2s;
}

.nav-brand:hover {
  color: #5a67d8;
}

.nav-links {
  display: flex;
  gap: 2rem;
}

.nav-link {
  color: #4a5568;
  text-decoration: none;
  font-weight: 600;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  transition: all 0.2s;
  position: relative;
}

.nav-link:hover {
  color: #667eea;
  background: rgba(102, 126, 234, 0.1);
}

.nav-link.router-link-active {
  color: #667eea;
  background: rgba(102, 126, 234, 0.15);
}

.main-content {
  padding: 2rem 0;
  min-height: calc(100vh - 80px);
}

@media (max-width: 640px) {
  .nav-container {
    padding: 1rem;
    flex-direction: column;
    gap: 1rem;
  }
  
  .nav-links {
    gap: 1rem;
  }
  
  .main-content {
    padding: 1rem 0;
  }
}
</style>
