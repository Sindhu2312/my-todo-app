<template>
  <div class="home">
    <TaskList />
  </div>
</template>

<script>
import TaskList from '../components/TaskList.vue';

export default {
  name: 'Home',
  components: {
    TaskList
  }
};
</script>

<style scoped>
.home {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}
</style>
