<script setup>
import TableDataComponent from './TableDataComponent.vue';
</script>

<template>
  <div class="p-5 bg-gray-100">
    <h1 class="text-3xl mb-5 font-medium">Transac</h1>

    The table displays a list of financial transactions, organized in a clean and responsive layout. It features five columns: Mobile (showing the phone number associated with each transaction), Amount (indicating the transaction value in currency), Date (displaying the transaction date in a readable format), Paybill (showing the paybill number linked to the transaction), and Type (specifying whether the transaction is a Deposit or Withdrawal). The table supports pagination, showing 10 transactions per page, with navigation buttons for Previous and Next pages. Filters above the table allow users to refine the data by date range, paybill, or search terms, with the default view covering the last 30 days. The design is user-friendly, with a clear, tabular structure styled using Tailwind CSS for a modern and professional appearance.


    <TableDataComponent/>
  </div>
</template>
