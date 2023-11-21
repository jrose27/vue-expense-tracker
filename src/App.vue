<template>
  <Header></Header>
  <div class="container">
    <Balance :total="+total"></Balance>
    <IncomeExpenses :income="+income" :expenses="+expenses"></IncomeExpenses>
    <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted"></TransactionList>
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted"></AddTransaction>
  </div>
</template>

<script setup>

import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

import { useToast } from 'vue-toastification';
import { ref,computed, onMounted } from 'vue';

const toast = useToast();

const transactions = ref([]);

onMounted( () => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactionsSaved'));

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

const total = computed(() => {
  return transactions.value.reduce((runningTotal, transaction) => {
    return runningTotal + transaction.amount;
  }, 0); 
});

// Get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((runningTotal, transaction) => {
      return runningTotal + transaction.amount;
    }, 0)
    .toFixed(2); 
});

// Get expenses
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((runningTotal, transaction) => {
      return runningTotal + transaction.amount;
    }, 0)
    .toFixed(2); 
});

// Add transaction
const handleTransactionSubmitted = (transactionData) => {
  //console.log(transactionData);
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount
  });
  //console.log(generateUniqueId());
  saveTransactionsToLocalStorage();
  toast.success(`${transactionData.text} ${transactionData.amount} Transaction Added!`)
};

// Generate unique id
const generateUniqueId = () => {
  return Math.floor(Math.random() * 100000);
};

// Delete transaction
const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter((transaction) => transaction.id !== id);
  saveTransactionsToLocalStorage();
  toast.success(`ID => ${id} Transaction Deleted!`);
};

// Save transactions to localstorage
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem('transactionsSaved', JSON.stringify(transactions.value));
};
</script>