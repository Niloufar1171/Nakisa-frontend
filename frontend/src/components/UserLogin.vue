
<template>
  <div>
    <h2>Login</h2>
    <form @submit.prevent="login">
      <input v-model="username" placeholder="Username" required />
      <input v-model="password" type="password" placeholder="Password" required />
      <button type="submit">Login</button>
    </form>
  </div>
</template>

<script>
//import axios from 'axios';

export default {
  data() {
    return {
      username: '',
      password: ''
    };
  },
  methods: {
    login() {
      localStorage.removeItem('auth');

      const credentials = btoa(`${this.username}:${this.password}`);
      localStorage.setItem('auth', credentials);
      this.$router.push('/expenses');
    }
  }
};
</script>
