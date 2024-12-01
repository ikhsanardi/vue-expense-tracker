<script setup>
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpenses from "./components/IncomeExpenses.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";
import { computed, onMounted, ref } from "vue";
import { useToast } from "vue-toastification";

const toast = useToast();
const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem("transactions"));

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

// Generate unique ID
const generateUniqueID = () => {
  return Math.floor(Math.random() * 1000000000);
};

// Get total
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});

// Get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Get expense
const expense = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// Add transaction
const handleTransactionSubmitted = (transaction) => {
  transactions.value.push({
    id: generateUniqueID(),
    text: transaction.text,
    amount: transaction.amount,
  });

  saveTransactionsToLocalStorage();
  toast.success("Transaction added");
};

// Delete transaction
const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter((transaction) => {
    return transaction.id !== id;
  });

  saveTransactionsToLocalStorage();
  toast.success("Transaction deleted");
};

// Save transactions to local storage
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem("transactions", JSON.stringify(transactions.value));
};
</script>

<template>
  <Header />

  <div class="container">
    <Balance :total="+total" />
    <IncomeExpenses :income="+income" :expense="+expense" />
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="handleTransactionDeleted"
    />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>
