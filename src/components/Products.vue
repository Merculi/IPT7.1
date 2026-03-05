<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const products = ref([])

// Funktion: Produkte laden
async function fetchProducts() {
  try {
    const res = await fetch("https://ipt71.kuno-schuerch.bbzwinf.ch/product/list")
    products.value = await res.json()
  } catch (err) {
    console.error("Ladefehler")
  }
}

// Funktion: Logout
function logout() {
  localStorage.removeItem('isLoggedIn')
  router.push('/')
}

onMounted(fetchProducts)
</script>

<template>
  <div class="products-page">
    <nav>
      <h1>beSmart Produkte</h1>
      <button @click="logout" class="logout-btn">Abmelden</button>
    </nav>

    <div class="grid">
      <div v-for="p in products" :key="p.id" class="p-card">
        <h3>{{ p.name }}</h3>
        <p>{{ p.price }} CHF</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
nav { display: flex; justify-content: space-between; align-items: center; background: #eee; padding: 10px 20px; }
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; padding: 20px; }
.p-card { border: 1px solid #ccc; padding: 15px; border-radius: 5px; }
.logout-btn { background: #ff4444; color: white; border: none; padding: 5px 15px; cursor: pointer; }
</style>