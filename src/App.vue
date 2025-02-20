<template>
  <Header />
  <div class="container">
    <Balance :total="total" />
    <IncomeExpense :income="+income" :expense="+expense" />
    <TransactionList :transactions="transactions" @transactionDelected="handleTransactionDeleted"/>
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
  import { ref, computed, onMounted } from 'vue';
  import AddTransaction from './components/AddTransaction.vue';
  import Balance from './components/Balance.vue';
  import Header from './components/Header.vue'
  import IncomeExpense from './components/IncomeExpense.vue';
  import TransactionList from './components/TransactionList.vue';

  import { useToast } from 'vue-toastification';

  const toast = useToast()

  const transactions = ref([]) 

  onMounted(() => {
    const savedTransactions = localStorage.getItem('transactions');
    
    if (savedTransactions) {
      try {
        transactions.value = JSON.parse(savedTransactions);
      } catch (error) {
        console.error("Error parsing transactions from localStorage:", error);
        transactions.value = []; // Reset transactions in case of bad data
      }
    }
  });


  // Get total
  const total = computed(() => {
    return transactions.value.reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
  })

  // Get income
  const income = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0).toFixed(2)
  })

  // Get expense
  const expense = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0).toFixed(2)
  })

  // Add transaction
  const handleTransactionSubmitted = (data) => {
    transactions.value.push({
      id: generateUniqueId(),
      text: data.text,
      amount: data.amount
    })

    saveTransactionsToLocalStorage()
    
    toast.success('Transaction added')
  }

  // Generate unique ID
  const generateUniqueId = () => {
    return Math.floor(Math.random() * 1000000)
  }

  // Delete transaction
  const handleTransactionDeleted = (id) => {
    transactions.value = transactions.value.filter(
      (transaction) => transaction.id !== id 
    );
    
    saveTransactionsToLocalStorage()

    toast.warning('Transaction deleted')
  }

  // Save to localstorage
  const saveTransactionsToLocalStorage = () => {
    localStorage.setItem('transactions', JSON.stringify(transactions.value));
  }
</script>

