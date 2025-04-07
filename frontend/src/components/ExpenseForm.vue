<template>
  <div>
    <h4>Add Expense</h4>
    <form @submit.prevent="addExpense">
      
      <input v-model="category" placeholder="category" required />
      <input v-model="amount" type="number" placeholder="Amount" required />
      <input v-model="date" type="date" required />
      <button type="submit">Add</button>
      <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
    </form>

    <!-- Filters Form -->
    <h4>Filter Expenses</h4>
    <form @submit.prevent="applyFilters">
      <input type="date" v-model="filters.startDate" placeholder="Start Date" />
      <input type="date" v-model="filters.endDate" placeholder="End Date" />
     <!-- Category Filter -->
      <input type="text" v-model="filters.category" placeholder="Category" />
       <!-- Amount Filters -->
      <input type="number" v-model="filters.minAmount" placeholder="Min Amount" />
      <input type="number" v-model="filters.maxAmount" placeholder="Max Amount" />
      
      <button type="submit">Apply Filters</button>
      <button type="button" @click="clearFilters">Clear</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      category: '',
      amount: 0,
      date: '',
      errorMessage: '',
      filters: {
        startDate: '',
        endDate: '',
        category: '',
        minAmount: '',
        maxAmount: ''
      }
    };
  },
  methods: {
    addExpense() {
      const auth = localStorage.getItem('auth');
      axios.post('http://localhost:8080/api/expenses', {
        category: this.category,
        amount: this.amount,
        date: this.date
      }, {
        headers: { 'Authorization': `Basic ${auth}` }
      })
      .then(() => {
        this.$emit('expense-added');
        this.category = '';
        this.amount = 0;
        this.date = '';
        this.errorMessage = '';
      })
      .catch(error => {
        if (error.response && error.response.data && error.response.data.message) {
          this.errorMessage = error.response.data.message;
        } else {
          this.errorMessage = 'An error occurred while adding the expense.';
        }
      });
    },

    applyFilters() {
      // Emit filters to parent
      this.$emit('filters-applied', { ...this.filters });
    },

    clearFilters() {
      // Reset filters
      this.filters = {
        startDate: '',
        endDate: '',
        category: '',
        minAmount: '',
        maxAmount: ''
      };
      this.$emit('filters-applied', { ...this.filters });
    }
  }
};
</script>
<style>
.error {
  color: red;
  margin-top: 10px;
}
</style>