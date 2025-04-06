<template>
  <div>
    <!-- Add Expense Form -->
    <h3>Add Expense</h3>
    <form @submit.prevent="addExpense">
      <input v-model="description" placeholder="Description" required />
      <input v-model="amount" type="number" placeholder="Amount" required />
      <input v-model="date" type="date" required />
      <button type="submit" :disabled="isLoading">Add</button>
    </form>
 <!-- Error Message Display -->
 <div v-if="error" class="error-message">
      <p>{{ error }}</p>
    </div>

    <!-- Filters Section -->
    <div class="filters">
      <label for="description">Description:</label>
      <input v-model="filters.description" id="description" type="text" placeholder="Enter description">

      <label for="startDate">Start Date:</label>
      <input v-model="filters.startDate" id="startDate" type="date">

      <label for="endDate">End Date:</label>
      <input v-model="filters.endDate" id="endDate" type="date">

      <label for="minAmount">Min Amount:</label>
      <input v-model="filters.minAmount" id="minAmount" type="number" min="0" step="0.01" placeholder="Min Amount">

      <label for="maxAmount">Max Amount:</label>
      <input v-model="filters.maxAmount" id="maxAmount" type="number" min="0" step="0.01" placeholder="Max Amount">

      <button type="submit" :disabled="isLoading">Apply Filters</button>
    </div>

    <!-- Expense Table Component -->
    <ExpenseTable :expenses="expenses" />

    <!-- Pagination Controls -->
    <div v-if="totalPages > 1" class="pagination">
      <button @click="changePage(page - 1)" :disabled="page <= 0">Previous</button>
      <span>Page {{ page + 1 }} of {{ totalPages }}</span>
      <button @click="changePage(page + 1)" :disabled="page >= totalPages - 1">Next</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import ExpenseTable from './ExpenseTable.vue'; // Import the ExpenseTable component

export default {
  components: {
    ExpenseTable,
  },
  data() {
    return {
      page: 0,
      size: 10,
      totalPages: 0,
      expenses: [],
      description: '',
      amount: 0,
      date: '',
      filters: {
        description: '',
        startDate: '',
        endDate: '',
        minAmount: null,
        maxAmount: null
      }
    };
  },
  methods: {
    // Fetch expenses from the backend with the selected filters
    getExpenses() {
      const { description, startDate, endDate, minAmount, maxAmount } = this.filters;
      const formattedStartDate = startDate ? startDate : null;
      const formattedEndDate = endDate ? endDate : null;
      axios
        .get('http://localhost:8080/api/expenses/filters', {
          params: {
            description: description || null,
            startDate: startDate || null,
            endDate: endDate || null,
            minAmount: minAmount || null,
            maxAmount: maxAmount || null,
            page: this.page,
            size: this.size
          },
          headers: {
        'Authorization': `Basic ${auth}` // Sending the Authorization header with the Basic auth token
      }
        })
        .then((response) => {
          this.expenses = response.data.content; // Page content
          this.totalPages = response.data.totalPages; // Total number of pages
        })
        .catch((error) => {
          console.error('Error fetching expenses:', error);
        });
    },
    // Change the current page for pagination
    changePage(newPage) {
      if (newPage >= 0 && newPage < this.totalPages) {
        this.page = newPage;
        this.getExpenses();
      }
    },
    // Add a new expense
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
        // Refresh expenses after adding a new one
        this.getExpenses();

        this.description = '';
        this.amount = 0;
        this.date = '';
      })
      .catch(error => console.error('Error adding expense:', error));
    }
  },
  mounted() {
    this.getExpenses(); // Fetch expenses when the component is mounted
  }
};
</script>

<style scoped>
.error-message {
  color: red;
  font-size: 14px;
  margin-top: 10px;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

table {
  width: 100%;
  margin-top: 20px;
  border-collapse: collapse;
}

table th, table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

table th {
  background-color: #f4f4f4;
}
</style>