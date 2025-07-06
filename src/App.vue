<template>
  <div>
    <nav>
      <router-link to="/home">Home</router-link> |
      <router-link to="/about">About</router-link> |
      <router-link to="/home">Home</router-link>
    </nav>
    <router-view />
  </div>
</template>

<script>
export default {};
</script>

<style>
nav {
  padding: 1rem;
  background-color: #eee;
}
nav a {
  margin-right: 1rem;
  text-decoration: none;
  color: blue;
}
</style>
