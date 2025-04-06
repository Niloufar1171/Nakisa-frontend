<template>
  <div>
    <h2>Add Expense</h2>
    <form @submit.prevent="addExpense">
      <input v-model="description" placeholder="Description" required />
      <input v-model="amount" type="number" placeholder="Amount" required />
      <input v-model="date" type="date" required />
      <button type="submit">Add</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      description: '',
      amount: 0,
      date: ''
    };
  },
  methods: {
    addExpense() {
      const auth = localStorage.getItem('auth');
      axios.post('http://localhost:8080/api/expenses', {
        description: this.description,
        amount: this.amount,
        date: this.date
      }, {
        headers: { 'Authorization': `Basic ${auth}` }
      })
      .then(() => {
        this.$emit('expense-added');
        this.description = '';
        this.amount = 0;
        this.date = '';
      })
      .catch(error => console.error(error));
    }
  }
};
</script>