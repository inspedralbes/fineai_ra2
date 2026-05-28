<script setup>
import { ref, onMounted, watch, computed } from 'vue'

const transactions = ref([])
const budgetLimit = ref(1000)
const newConcept = ref('')
const newAmount = ref(null)
const newType = ref('expense')
const newCategory = ref('oci')
const isBudgetEditing = ref(false)
const tempBudget = ref(1000)

// Carregar dades de LocalStorage en muntar
onMounted(() => {
  if (process.client) {
    const savedTransactions = localStorage.getItem('fineai_transactions')
    if (savedTransactions) {
      transactions.value = JSON.parse(savedTransactions)
    }

    const savedBudget = localStorage.getItem('fineai_budget')
    if (savedBudget) {
      budgetLimit.value = parseFloat(savedBudget)
      tempBudget.value = budgetLimit.value
    } else {
      localStorage.setItem('fineai_budget', budgetLimit.value.toString())
    }
  }
})

// Observar transaccions per desar canvis
watch(transactions, (newVal) => {
  if (process.client) {
    localStorage.setItem('fineai_transactions', JSON.stringify(newVal))
  }
}, { deep: true })

// Calcular totals
const totalIncome = computed(() => {
  return transactions.value
    .filter(t => t.type === 'income')
    .reduce((sum, t) => sum + t.amount, 0)
})

const totalExpense = computed(() => {
  return transactions.value
    .filter(t => t.type === 'expense')
    .reduce((sum, t) => sum + t.amount, 0)
})

const netBalance = computed(() => {
  return totalIncome.value - totalExpense.value
})

const expensePercentage = computed(() => {
  if (budgetLimit.value <= 0) return 0
  const percentage = (totalExpense.value / budgetLimit.value) * 100
  return Math.min(Math.round(percentage), 100)
})

// Mètodes
function addTransaction() {
  if (!newConcept.value || !newAmount.value || newAmount.value <= 0) return

  const transaction = {
    id: Date.now().toString(),
    concept: newConcept.value,
    amount: parseFloat(newAmount.value),
    type: newType.value,
    category: newCategory.value,
    date: new Date().toISOString()
  }

  transactions.value.unshift(transaction)

  // Reset formulari
  newConcept.value = ''
  newAmount.value = null
}

function deleteTransaction(id) {
  transactions.value = transactions.value.filter(t => t.id !== id)
}

function saveBudget() {
  if (tempBudget.value && tempBudget.value > 0) {
    budgetLimit.value = parseFloat(tempBudget.value)
    if (process.client) {
      localStorage.setItem('fineai_budget', budgetLimit.value.toString())
    }
    isBudgetEditing.value = false
  }
}
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <div class="logo-area">
        <h1 class="logo-title">FineAI</h1>
        <span class="logo-subtitle">Gestor Intel·ligent de Finances</span>
      </div>
      <div class="budget-control">
        <div v-if="!isBudgetEditing" class="budget-display">
          <span class="budget-label">Límit mensual:</span>
          <span class="budget-value">{{ budgetLimit.toFixed(2) }} EUR</span>
          <button @click="isBudgetEditing = true" class="btn-edit">Editar</button>
        </div>
        <div v-else class="budget-form">
          <input 
            type="number" 
            v-model="tempBudget" 
            class="input-budget" 
            min="1"
          />
          <button @click="saveBudget" class="btn-save">Desar</button>
        </div>
      </div>
    </header>

    <main class="app-main">
      <!-- Secció superior: Indicadors de Saldo i Gràfic de Pressupost -->
      <section class="dashboard-widgets">
        <div class="widget balance-card">
          <h3>Balanç Net</h3>
          <p class="balance-value" :class="{ 'negative-val': netBalance < 0 }">
            {{ netBalance.toFixed(2) }} EUR
          </p>
        </div>

        <div class="widget-row">
          <div class="widget stats-card income-bg">
            <span class="stats-label">Ingressos totals</span>
            <span class="stats-value text-green">{{ totalIncome.toFixed(2) }} EUR</span>
          </div>
          <div class="widget stats-card expense-bg">
            <span class="stats-label">Despeses totals</span>
            <span class="stats-value text-red">{{ totalExpense.toFixed(2) }} EUR</span>
          </div>
        </div>

        <div class="widget budget-progress-card">
          <div class="budget-meta">
            <span>Ús del pressupost mensual</span>
            <span>{{ expensePercentage }}%</span>
          </div>
          <div class="progress-bar-container">
            <div 
              class="progress-bar-fill" 
              :style="{ width: expensePercentage + '%' }"
              :class="{ 'alert-bar': expensePercentage >= 90 }"
            ></div>
          </div>
          <p class="progress-subtitle">
            Has gastat {{ totalExpense.toFixed(2) }} EUR de {{ budgetLimit.toFixed(2) }} EUR límit.
          </p>
        </div>
      </section>

      <!-- Secció central: Dues columnes adaptatives -->
      <section class="main-content-grid">
        <!-- Columna Esquerra: Formulari minimalista d'entrada -->
        <div class="form-container">
          <h2 class="section-title">Afegir Transacció</h2>
          <form @submit.prevent="addTransaction" class="transaction-form">
            <div class="form-group">
              <label for="concept">Concepte</label>
              <input 
                id="concept"
                type="text" 
                v-model="newConcept" 
                placeholder="Compra Mercadona, Nòmina..." 
                required
                class="form-input"
              />
            </div>

            <div class="form-group">
              <label for="amount">Import (EUR)</label>
              <input 
                id="amount"
                type="number" 
                v-model="newAmount" 
                step="0.01"
                placeholder="0.00" 
                required
                class="form-input"
                min="0.01"
              />
            </div>

            <div class="form-group">
              <label>Tipus</label>
              <div class="radio-group">
                <label class="radio-label">
                  <input 
                    type="radio" 
                    v-model="newType" 
                    value="expense" 
                  />
                  <span>Despesa</span>
                </label>
                <label class="radio-label">
                  <input 
                    type="radio" 
                    v-model="newType" 
                    value="income" 
                  />
                  <span>Ingrés</span>
                </label>
              </div>
            </div>

            <div class="form-group">
              <label for="category">Categoria</label>
              <select 
                id="category"
                v-model="newCategory" 
                class="form-select"
              >
                <option value="alimentacio">Alimentació</option>
                <option value="transport">Transport</option>
                <option value="habitatge">Habitatge</option>
                <option value="oci">Oci i Espectacles</option>
                <option value="factures">Factures i Serveis</option>
                <option value="nomina" v-if="newType === 'income'">Nòmina i Treball</option>
                <option value="altres">Altres</option>
              </select>
            </div>

            <button type="submit" class="btn-submit">Afegir</button>
          </form>
        </div>

        <!-- Columna Dreta: Llista de transaccions mitjançant component importat -->
        <div class="list-container">
          <h2 class="section-title">Transaccions Recents</h2>
          <TransactionList 
            :transactions="transactions" 
            @delete-transaction="deleteTransaction" 
          />
        </div>
      </section>
    </main>

    <!-- Assistent Financer Flotant -->
    <AICoachChat 
      :transactions="transactions" 
      :budgetLimit="budgetLimit" 
    />
  </div>
</template>

<style>
:root {
  --bg-primary: #0B0F19;
  --card-bg: rgba(20, 27, 45, 0.6);
  --accent-green: #10B981;
  --accent-red: #EF4444;
  --accent-cyan: #06B6D4;
  --text-main: #F3F4F6;
  --text-muted: #9CA3AF;
  --glass-border: 1px solid rgba(255, 255, 255, 0.08);
  --glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
  --font-title: 'Outfit', sans-serif;
  --font-body: 'Plus Jakarta Sans', sans-serif;
}

body {
  margin: 0;
  padding: 0;
  background-color: var(--bg-primary);
  color: var(--text-main);
  font-family: var(--font-body);
  overflow-x: hidden;
  background-image: radial-gradient(circle at 50% 30%, #17223F 0%, #0B0F19 80%);
  background-attachment: fixed;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  box-sizing: border-box;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background: var(--card-bg);
  backdrop-filter: blur(12px);
  border: var(--glass-border);
  border-radius: 16px;
  box-shadow: var(--glass-shadow);
  margin-bottom: 25px;
}

.logo-area {
  display: flex;
  flex-direction: column;
}

.logo-title {
  margin: 0;
  font-family: var(--font-title);
  font-size: 2.2rem;
  font-weight: 700;
  letter-spacing: -1px;
  background: linear-gradient(to right, var(--text-main), var(--accent-cyan));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.logo-subtitle {
  font-size: 0.85rem;
  color: var(--text-muted);
  font-weight: 500;
}

.budget-display, .budget-form {
  display: flex;
  align-items: center;
  gap: 10px;
}

.budget-label {
  font-size: 0.9rem;
  color: var(--text-muted);
}

.budget-value {
  font-family: var(--font-title);
  font-weight: 700;
  color: var(--accent-cyan);
}

.btn-edit, .btn-save {
  background: rgba(255, 255, 255, 0.05);
  border: var(--glass-border);
  color: var(--text-main);
  padding: 6px 12px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.8rem;
  transition: all 0.3s ease;
}

.btn-edit:hover, .btn-save:hover {
  background: var(--accent-cyan);
  color: var(--bg-primary);
  font-weight: 600;
}

.input-budget {
  background: rgba(0, 0, 0, 0.2);
  border: var(--glass-border);
  color: var(--text-main);
  padding: 6px;
  border-radius: 8px;
  width: 100px;
  outline: none;
}

.app-main {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

.dashboard-widgets {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
}

@media (min-width: 768px) {
  .dashboard-widgets {
    grid-template-columns: 1.2fr 2fr;
  }
}

.widget {
  background: var(--card-bg);
  backdrop-filter: blur(12px);
  border: var(--glass-border);
  border-radius: 16px;
  box-shadow: var(--glass-shadow);
  padding: 24px;
}

.balance-card {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.balance-card h3 {
  margin: 0 0 10px 0;
  font-size: 1.1rem;
  color: var(--text-muted);
  font-weight: 500;
}

.balance-value {
  margin: 0;
  font-family: var(--font-title);
  font-size: 2.5rem;
  font-weight: 700;
  color: var(--accent-green);
}

.balance-value.negative-val {
  color: var(--accent-red);
}

.widget-row {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

@media (min-width: 480px) {
  .widget-row {
    flex-direction: row;
  }
}

.stats-card {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.stats-label {
  font-size: 0.9rem;
  color: var(--text-muted);
}

.stats-value {
  font-family: var(--font-title);
  font-size: 1.6rem;
  font-weight: 700;
}

.text-green { color: var(--accent-green); }
.text-red { color: var(--accent-red); }

.budget-progress-card {
  grid-column: 1 / -1;
}

.budget-meta {
  display: flex;
  justify-content: space-between;
  font-size: 0.95rem;
  margin-bottom: 10px;
  font-weight: 600;
}

.progress-bar-container {
  height: 10px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 5px;
  overflow: hidden;
  margin-bottom: 12px;
  border: 1px solid rgba(255, 255, 255, 0.03);
}

.progress-bar-fill {
  height: 100%;
  background: linear-gradient(to right, var(--accent-cyan), var(--accent-green));
  border-radius: 5px;
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.progress-bar-fill.alert-bar {
  background: linear-gradient(to right, var(--accent-red), #F59E0B);
}

.progress-subtitle {
  margin: 0;
  font-size: 0.85rem;
  color: var(--text-muted);
}

.main-content-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 25px;
}

@media (min-width: 992px) {
  .main-content-grid {
    grid-template-columns: 1fr 1.6fr;
  }
}

.form-container, .list-container {
  background: var(--card-bg);
  backdrop-filter: blur(12px);
  border: var(--glass-border);
  border-radius: 16px;
  box-shadow: var(--glass-shadow);
  padding: 24px;
}

.section-title {
  margin: 0 0 20px 0;
  font-family: var(--font-title);
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-main);
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 10px;
}

.transaction-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-muted);
}

.form-input, .form-select {
  background: rgba(0, 0, 0, 0.2);
  border: var(--glass-border);
  color: var(--text-main);
  padding: 10px 14px;
  border-radius: 10px;
  font-family: var(--font-body);
  font-size: 0.9rem;
  outline: none;
  transition: all 0.3s ease;
}

.form-input:focus, .form-select:focus {
  border-color: var(--accent-cyan);
  box-shadow: 0 0 10px rgba(6, 182, 212, 0.15);
}

.radio-group {
  display: flex;
  gap: 20px;
  padding: 5px 0;
}

.radio-label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  cursor: pointer;
}

.radio-label input {
  accent-color: var(--accent-cyan);
}

.btn-submit {
  background: var(--accent-cyan);
  color: var(--bg-primary);
  border: none;
  padding: 12px;
  border-radius: 10px;
  font-family: var(--font-title);
  font-weight: 700;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  margin-top: 10px;
}

.btn-submit:hover {
  background: #22D3EE;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(6, 182, 212, 0.3);
}

.btn-submit:active {
  transform: translateY(0);
}
</style>
