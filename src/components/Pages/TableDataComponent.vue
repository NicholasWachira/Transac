<template>
  <div class="container mx-auto p-4">
    <!-- Tabs -->
    <div class="flex border-b mb-4">
      <button
        v-for="tab in tabs"
        :key="tab"
        @click="activeTab = tab"
        :class="[
          'px-4 py-2 font-medium',
          activeTab === tab ? 'border-b-2 border-blue-500 text-blue-600' : 'text-gray-600'
        ]"
      >
        {{ tab }}
      </button>
    </div>

    <!-- Filters -->
    <div class="flex flex-col md:flex-row gap-4 mb-4">
      <!-- Date Filters -->
      <div class="flex gap-4">
        <div>
          <label class="block text-sm font-medium text-gray-700">Start Date</label>
          <input
            type="date"
            v-model="filters.startDate"
            @change="applyFilters"
            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700">End Date</label>
          <input
            type="date"
            v-model="filters.endDate"
            @change="applyFilters"
            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700">Summary Date</label>
          <input
            type="date"
            v-model="filters.summaryDate"
            @change="applyFilters"
            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
          />
        </div>
      </div>

      <!-- Paybill Filter -->
      <div>
        <label class="block text-sm font-medium text-gray-700">Paybill</label>
        <select
          v-model="filters.paybill"
          @change="applyFilters"
          class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
        >
          <option value="">All Paybills</option>
          <option v-for="paybill in uniquePaybills" :key="paybill" :value="paybill">
            {{ paybill }}
          </option>
        </select>
      </div>

      <!-- Search Bar with Dropdown -->
      <div class="flex-1">
        <label class="block text-sm font-medium text-gray-700">Search Transactions</label>
        <div class="flex mt-1">
          <select
            v-model="searchField"
            class="rounded-l-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
          >
            <option value="mobile">Mobile</option>
            <option value="amount">Amount</option>
            <option value="paybill">Paybill</option>
          </select>
          <input
            type="text"
            v-model="searchQuery"
            @input="applyFilters"
            placeholder="Search..."
            class="flex-1 rounded-md border border-gray-800 shadow-sm p-1"
          />
        </div>
      </div>
    </div>

    <!-- Summary Statistics -->
    <div class="bg-gray-100 p-4 rounded-md mb-4">
      <h3 class="text-lg font-semibold text-gray-800 mb-2">Transaction Summary</h3>
      <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
        <div>
          <p class="text-sm text-gray-600">Total Deposits</p>
          <p class="text-xl font-bold text-green-600">{{ totalDepositsAmount }}</p>
        </div>
        <div>
          <p class="text-sm text-gray-600">Deposit Count</p>
          <p class="text-xl font-bold">{{ depositCount }}</p>
        </div>
        <div>
          <p class="text-sm text-gray-600">Total Withdrawals</p>
          <p class="text-xl font-bold text-red-600">{{ totalWithdrawalsAmount }}</p>
        </div>
        <div>
          <p class="text-sm text-gray-600">Withdrawal Count</p>
          <p class="text-xl font-bold">{{ withdrawalCount }}</p>
        </div>
      </div>
    </div>

    <!-- Transactions Table -->
    <div class="overflow-x-auto">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
          <tr>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Mobile</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Amount</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Date</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Paybill</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Type</th>
          </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
          <tr v-for="transaction in paginatedTransactions" :key="transaction.id">
            <td class="px-6 py-4 whitespace-nowrap">{{ transaction.mobile }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ transaction.amount }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ formatDate(transaction.date) }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ transaction.paybill }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ transaction.type }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div class="flex justify-between items-center mt-4">
      <div>
        Showing {{ (currentPage - 1) * itemsPerPage + 1 }} to
        {{ Math.min(currentPage * itemsPerPage, filteredTransactions.length) }} of
        {{ filteredTransactions.length }} transactions
      </div>
      <div class="flex gap-2">
        <button
          @click="currentPage--"
          :disabled="currentPage === 1"
          class="px-4 py-2 bg-gray-200 rounded disabled:opacity-50"
        >
          Previous
        </button>
        <button
          @click="currentPage++"
          :disabled="currentPage * itemsPerPage >= filteredTransactions.length"
          class="px-4 py-2 bg-gray-200 rounded disabled:opacity-50"
        >
          Next
        </button>
      </div>
    </div>

    <!-- Daily Summary Table -->
    <div class="mt-6">
      <h3 class="text-lg font-semibold text-gray-800 mb-2">Daily Summary for {{ filters.summaryDate || 'Selected Date' }}</h3>
      <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-gray-200">
          <thead class="bg-gray-50">
            <tr>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Date</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Deposit Count</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Deposit Amount</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Withdrawal Count</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Withdrawal Amount</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Net Cash</th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200">
            <tr v-if="filters.summaryDate">
              <td class="px-6 py-4 whitespace-nowrap">{{ formatDate(filters.summaryDate) }}</td>
              <td class="px-6 py-4 whitespace-nowrap">{{ dailyDepositCount }}</td>
              <td class="px-6 py-4 whitespace-nowrap text-green-600">{{ dailyDepositAmount }}</td>
              <td class="px-6 py-4 whitespace-nowrap">{{ dailyWithdrawalCount }}</td>
              <td class="px-6 py-4 whitespace-nowrap text-red-600">{{ dailyWithdrawalAmount }}</td>
              <td class="px-6 py-4 whitespace-nowrap" :class="dailyNetCash >= 0 ? 'text-green-600' : 'text-red-600'">
                {{ dailyNetCash }}
              </td>
            </tr>
            <tr v-else>
              <td colspan="6" class="px-6 py-4 text-center text-gray-500">Select a date to view summary</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TransactionPage',
  data() {
    return {
      activeTab: 'All',
      tabs: ['All', 'Deposits', 'Withdrawals'],
      filters: {
        startDate: '',
        endDate: '',
        paybill: '',
        summaryDate: '', // New filter for summary table date
      },
      searchQuery: '',
      searchField: 'mobile',
      currentPage: 1,
      itemsPerPage: 10,
      transactions: [
        { id: 1, mobile: '0712345678', amount: 500, date: '2025-06-01', paybill: '123456', type: 'Deposit' },
        { id: 2, mobile: '0723456789', amount: 1000, date: '2025-06-02', paybill: '123456', type: 'Withdrawal' },
        { id: 3, mobile: '0734567890', amount: 750, date: '2025-06-03', paybill: '789012', type: 'Deposit' },
        { id: 4, mobile: '0745678901', amount: 200, date: '2025-06-04', paybill: '123456', type: 'Withdrawal' },
        { id: 5, mobile: '0756789012', amount: 1500, date: '2025-06-05', paybill: '456789', type: 'Deposit' },
        { id: 6, mobile: '0767890123', amount: 300, date: '2025-06-06', paybill: '789012', type: 'Withdrawal' },
        { id: 7, mobile: '0778901234', amount: 800, date: '2025-06-07', paybill: '123456', type: 'Deposit' },
        { id: 8, mobile: '0789012345', amount: 1200, date: '2025-06-08', paybill: '456789', type: 'Withdrawal' },
        { id: 9, mobile: '0790123456', amount: 600, date: '2025-06-09', paybill: '789012', type: 'Deposit' },
        { id: 10, mobile: '0701234567', amount: 900, date: '2025-06-09', paybill: '123456', type: 'Withdrawal' },
        { id: 11, mobile: '0712345678', amount: 1100, date: '2025-06-09', paybill: '456789', type: 'Deposit' },
        { id: 12, mobile: '0723456789', amount: 400, date: '2025-06-09', paybill: '789012', type: 'Withdrawal' },
        { id: 13, mobile: '0734567890', amount: 700, date: '2025-06-09', paybill: '123456', type: 'Deposit' },
        { id: 14, mobile: '0745678901', amount: 1300, date: '2025-06-09', paybill: '456789', type: 'Withdrawal' },
        { id: 15, mobile: '0756789012', amount: 2000, date: '2025-06-15', paybill: '789012', type: 'Deposit' },
        { id: 16, mobile: '0767890123', amount: 500, date: '2025-06-16', paybill: '123456', type: 'Withdrawal' },
        { id: 17, mobile: '0778901234', amount: 950, date: '2025-06-17', paybill: '456789', type: 'Deposit' },
        { id: 18, mobile: '0789012345', amount: 300, date: '2025-06-18', paybill: '789012', type: 'Withdrawal' },
        { id: 19, mobile: '0790123456', amount: 850, date: '2025-06-19', paybill: '123456', type: 'Deposit' },
        { id: 20, mobile: '0701234567', amount: 1400, date: '2025-06-20', paybill: '456789', type: 'Withdrawal' },
      ],
    };
  },
  computed: {
    uniquePaybills() {
      return [...new Set(this.transactions.map(t => t.paybill))];
    },
    filteredTransactions() {
      let result = this.transactions;

      // Filter by tab
      if (this.activeTab === 'Deposits') {
        result = result.filter(t => t.type === 'Deposit');
      } else if (this.activeTab === 'Withdrawals') {
        result = result.filter(t => t.type === 'Withdrawal');
      }

      // Filter by date
      if (this.filters.startDate) {
        result = result.filter(t => t.date >= this.filters.startDate);
      }
      if (this.filters.endDate) {
        result = result.filter(t => t.date <= this.filters.endDate);
      }

      // Filter by paybill
      if (this.filters.paybill) {
        result = result.filter(t => t.paybill === this.filters.paybill);
      }

      // Search
      if (this.searchQuery) {
        result = result.filter(t =>
          String(t[this.searchField]).toLowerCase().includes(this.searchQuery.toLowerCase())
        );
      }

      return result;
    },
    paginatedTransactions() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.filteredTransactions.slice(start, end);
    },
    totalDepositsAmount() {
      return this.filteredTransactions
        .filter(t => t.type === 'Deposit')
        .reduce((sum, t) => sum + t.amount, 0);
    },
    totalWithdrawalsAmount() {
      return this.filteredTransactions
        .filter(t => t.type === 'Withdrawal')
        .reduce((sum, t) => sum + t.amount, 0);
    },
    depositCount() {
      return this.filteredTransactions.filter(t => t.type === 'Deposit').length;
    },
    withdrawalCount() {
      return this.filteredTransactions.filter(t => t.type === 'Withdrawal').length;
    },
    dailyDepositCount() {
      if (!this.filters.summaryDate) return 0;
      return this.transactions
        .filter(t => t.date === this.filters.summaryDate && t.type === 'Deposit')
        .length;
    },
    dailyDepositAmount() {
      if (!this.filters.summaryDate) return 0;
      return this.transactions
        .filter(t => t.date === this.filters.summaryDate && t.type === 'Deposit')
        .reduce((sum, t) => sum + t.amount, 0);
    },
    dailyWithdrawalCount() {
      if (!this.filters.summaryDate) return 0;
      return this.transactions
        .filter(t => t.date === this.filters.summaryDate && t.type === 'Withdrawal')
        .length;
    },
    dailyWithdrawalAmount() {
      if (!this.filters.summaryDate) return 0;
      return this.transactions
        .filter(t => t.date === this.filters.summaryDate && t.type === 'Withdrawal')
        .reduce((sum, t) => sum + t.amount, 0);
    },
    dailyNetCash() {
      return this.dailyDepositAmount - this.dailyWithdrawalAmount;
    },
  },
  methods: {
    applyFilters() {
      this.currentPage = 1; // Reset to first page when filters change
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString();
    },
    setDefaultDateRange() {
      const end = new Date('2025-07-01');
      const start = new Date();
      start.setDate(end.getDate() - 30);
      this.filters.startDate = start.toISOString().split('T')[0];
      this.filters.endDate = end.toISOString().split('T')[0];
      this.filters.summaryDate = end.toISOString().split('T')[0]; // Set default summary date
    },
  },
  mounted() {
    this.setDefaultDateRange();
  },
};
</script>

<style scoped>
/* Tailwind handles most styling, but we can add custom styles if needed */
</style>