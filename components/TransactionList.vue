<script setup>
import { computed } from 'vue'

const props = defineProps({
  transactions: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['delete-transaction'])

function formatCategory(category) {
  const map = {
    alimentacio: 'Alimentació',
    transport: 'Transport',
    habitatge: 'Habitatge',
    oci: 'Oci',
    factures: 'Factures',
    nomina: 'Nòmina',
    altres: 'Altres'
  }
  return map[category] || category
}

function formatDate(isoString) {
  const date = new Date(isoString)
  return date.toLocaleDateString('ca-ES', {
    day: '2-digit',
    month: 'short',
    hour: '2-digit',
    minute: '2-digit'
  })
}
</script>

<template>
  <div class="transactions-wrapper">
    <div v-if="transactions.length === 0" class="empty-state">
      <p class="empty-text">No hi ha transaccions registrades.</p>
      <p class="empty-subtext">Comença per afegir la teva primera despesa o ingrés.</p>
    </div>

    <div v-else class="transactions-list">
      <div 
        v-for="item in transactions" 
        :key="item.id" 
        class="transaction-card"
        :class="item.type"
      >
        <div class="card-left">
          <div class="card-details">
            <span class="card-concept">{{ item.concept }}</span>
            <span class="card-date">{{ formatDate(item.date) }}</span>
          </div>
        </div>

        <div class="card-right">
          <div class="card-meta">
            <span 
              class="card-amount" 
              :class="item.type === 'income' ? 'income-text' : 'expense-text'"
            >
              {{ item.type === 'income' ? '+' : '-' }}{{ item.amount.toFixed(2) }} EUR
            </span>
            <span class="category-tag">{{ formatCategory(item.category) }}</span>
          </div>
          
          <button 
            @click="emit('delete-transaction', item.id)" 
            class="btn-delete"
            aria-label="Eliminar transacció"
          >
            Eliminar
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.transactions-wrapper {
  max-height: 480px;
  overflow-y: auto;
  padding-right: 5px;
}

/* Scrollbar personalitzat */
.transactions-wrapper::-webkit-scrollbar {
  width: 6px;
}
.transactions-wrapper::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.02);
  border-radius: 3px;
}
.transactions-wrapper::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}
.transactions-wrapper::-webkit-scrollbar-thumb:hover {
  background: var(--accent-cyan);
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
}

.empty-text {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--text-main);
  margin: 0 0 8px 0;
}

.empty-subtext {
  font-size: 0.85rem;
  color: var(--text-muted);
  margin: 0;
}

.transactions-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.transaction-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.04);
  border-radius: 12px;
  padding: 14px 18px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.transaction-card:hover {
  background: rgba(255, 255, 255, 0.04);
  transform: translateX(4px);
  border-color: rgba(255, 255, 255, 0.08);
}

.transaction-card.income {
  border-left: 4px solid var(--accent-green);
}

.transaction-card.expense {
  border-left: 4px solid var(--accent-red);
}

.card-left {
  display: flex;
  align-items: center;
  gap: 15px;
}

.card-details {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.card-concept {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-main);
}

.card-date {
  font-size: 0.75rem;
  color: var(--text-muted);
}

.card-right {
  display: flex;
  align-items: center;
  gap: 20px;
}

.card-meta {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 6px;
}

.card-amount {
  font-family: var(--font-title);
  font-size: 1.05rem;
  font-weight: 700;
}

.income-text {
  color: var(--accent-green);
}

.expense-text {
  color: var(--accent-red);
}

.category-tag {
  font-size: 0.75rem;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.04);
  color: var(--text-muted);
  padding: 2px 8px;
  border-radius: 6px;
  font-weight: 500;
}

.btn-delete {
  background: transparent;
  border: 1px solid rgba(239, 68, 68, 0.2);
  color: var(--accent-red);
  padding: 6px 10px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.75rem;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-delete:hover {
  background: var(--accent-red);
  color: var(--text-main);
  border-color: var(--accent-red);
  box-shadow: 0 0 10px rgba(239, 68, 68, 0.2);
}
</style>
