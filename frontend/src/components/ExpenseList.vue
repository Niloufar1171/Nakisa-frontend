<template>
  <div>
    <h2>Expenses</h2>
    <expense-form @expense-added="fetchExpenses" />
    <ul>
      <li v-for="expense in expenses" :key="expense.id">
        {{ expense.description }} - {{ expense.amount }} - {{ expense.date }}
        <button @click="deleteExpense(expense.id)">Delete</button>
      </li>
    </ul>
    <button @click="logout">Logout</button>
  </div>
</template>

<script>
import axios from 'axios';
import ExpenseForm from './ExpenseForm.vue';

export default {
  components: { ExpenseForm },
  data() {
    return {
      expenses: []
    };
  },
  mounted() {
    this.fetchExpenses();
  },
  methods: {
    fetchExpenses() {
      const auth = localStorage.getItem('auth');
      if (!auth) {
        this.$router.push('/');
        return;
      }
      axios.get('http://localhost:8080/api/expenses', {
        headers: { 'Authorization': `Basic ${auth}` }
      })
      .then(response => {
        this.expenses = response.data;
      })
      .catch(() => this.$router.push('/'));
    },
    deleteExpense(id) {
      const auth = localStorage.getItem('auth');
      axios.delete(`http://localhost:8080/api/expenses/${id}`, {
        headers: { 'Authorization': `Basic ${auth}` }
      })
      .then(() => this.fetchExpenses());
    },
    logout() {
      localStorage.removeItem('auth');
      this.$router.push('/');
    }
  }
};
</script>