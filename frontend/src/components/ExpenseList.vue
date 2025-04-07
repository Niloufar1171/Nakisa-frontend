<template>
  <div>
    <h2>Expenses</h2>

    <!-- Include the Expense Form (where you add expenses) -->
    <expense-form @expense-added="fetchExpenses" @filters-applied="applyFilters" />

    <!-- Loader (Show when data is loading) -->
    <div v-if="isLoading">Loading...</div>

    <!-- Display List of Expenses -->
    <ul>
      <li v-for="expense in expenses" :key="expense.id">
        {{ expense.category }} - {{ expense.amount }} - {{ expense.date }} - {{ expense.category }}
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
      expenses: [],
      filters: {
        category: '',
        startDate: '',
        endDate: '',
        minAmount: '',
        maxAmount: ''
      },
      isLoading: false // Loader state for data fetching
    };
  },
  mounted() {
    this.fetchExpenses();
  },
  methods: {
    fetchExpenses() {
      this.isLoading = true; // Show loader when fetching
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
      .catch(() => this.$router.push('/'))
      .finally(() => {
        this.isLoading = false; // Hide loader after fetching is complete
      });
    },

    // Called when filters are applied (through event)
    applyFilters(filters) {
      this.isLoading = true; // Show loader when fetching filtered results
      const auth = localStorage.getItem('auth');
      axios.get('http://localhost:8080/api/expenses/filters', {
        headers: { 'Authorization': `Basic ${auth}` },
        params: {
          ...filters,
          page: 0,
          size: 10
        }
      })
      .then(response => {
        this.expenses = response.data.content || response.data;
      })
      .catch(error => console.error('Filter failed', error))
      .finally(() => {
        this.isLoading = false; // Hide loader after filtering is complete
      });
    },

    // Delete an expense
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
