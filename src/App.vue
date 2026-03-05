<script setup>
import { ref, reactive } from 'vue'

// --- APP ZUSTAND ---
const currentView = ref('register') // Wechselt zwischen 'register', 'login' und 'products'
const products = ref([])
const API_BASE = "https://ipt71.kuno-schuerch.bbzwinf.ch"

// --- DATEN MODELLE ---
const regForm = reactive({
  username: '',   // Muss E-Mail sein laut PDF
  firstname: '',
  lastname: '',
  password: '',
  language: 'de'  // Enum: de, en, fr
})

const loginForm = reactive({
  username: '',
  password: ''
})

// --- LOGIK & VALIDIERUNG ---

// Validierung vor dem Absenden (Anforderung PDF Seite 1)
function isFormValid() {
  if (!regForm.username.includes('@')) {
    alert("Benutzername muss eine gültige E-Mail Adresse sein.")
    return false
  }
  if (regForm.password.length < 4) {
    alert("Kennwort ist zu kurz.")
    return false
  }
  return true
}

// 1. REGISTRIERUNG
async function handleRegister() {
  if (!isFormValid()) return

  try {
    const response = await fetch(`${API_BASE}/user/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(regForm)
    })

    if (response.ok) {
      alert("Registrierung erfolgreich! Bitte melden Sie sich an.")
      currentView.value = 'login'
    } else {
      alert("Fehler bei der Registrierung. Eventuell existiert der Benutzer schon.")
    }
  } catch (error) {
    alert("Backend nicht erreichbar.")
  }
}

// 2. LOGIN
async function handleLogin() {
  try {
    const response = await fetch(`${API_BASE}/user/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(loginForm)
    })

    if (response.ok) {
      // Wenn Login erfolgreich, lade Produkte und wechsle Ansicht
      await fetchProducts()
      currentView.value = 'products'
    } else {
      alert("Anmeldung fehlgeschlagen. Passwort oder E-Mail falsch.")
    }
  } catch (error) {
    alert("Login-Fehler.")
  }
}

// 3. PRODUKTE LADEN
async function fetchProducts() {
  try {
    const response = await fetch(`${API_BASE}/product/list`)
    if (response.ok) {
      products.value = await response.json()
    }
  } catch (error) {
    console.error("Produkte konnten nicht geladen werden.")
  }
}

// 4. LOGOUT (Anforderung PDF Seite 1)
function handleLogout() {
  currentView.value = 'login'
  // Passwörter leeren aus Sicherheitsgründen
  regForm.password = ''
  loginForm.password = ''
}
</script>

<template>
  <div class="app-container">

    <!-- ANSICHT: REGISTRATION -->
    <div v-if="currentView === 'register'" class="card">
      <h2>beSmart - Registrierung</h2>
      <form @submit.prevent="handleRegister">
        <input v-model="regForm.username" type="email" placeholder="Benutzername (E-Mail)" required>
        <input v-model="regForm.firstname" type="text" placeholder="Vorname" required>
        <input v-model="regForm.lastname" type="text" placeholder="Nachname" required>
        <input v-model="regForm.password" type="password" placeholder="Kennwort" required>

        <label>Gewünschte Sprache:</label>
        <select v-model="regForm.language">
          <option value="de">Deutsch</option>
          <option value="en">Englisch</option>
          <option value="fr">Französisch</option>
        </select>

        <button type="submit" class="primary-btn">Registrieren</button>
      </form>
      <p @click="currentView = 'login'" class="switch-link">Bereits registriert? Zum Login</p>
    </div>

    <!-- ANSICHT: LOGIN -->
    <div v-else-if="currentView === 'login'" class="card">
      <h2>beSmart - Anmeldung</h2>
      <form @submit.prevent="handleLogin">
        <input v-model="loginForm.username" type="email" placeholder="E-Mail" required>
        <input v-model="loginForm.password" type="password" placeholder="Kennwort" required>
        <button type="submit" class="primary-btn">Einloggen</button>
      </form>
      <p @click="currentView = 'register'" class="switch-link">Noch kein Konto? Hier registrieren</p>
    </div>

    <!-- ANSICHT: PRODUKTSEITE -->
    <div v-else-if="currentView === 'products'" class="product-page">
      <header class="header">
        <h1>Unsere Produkte</h1>
        <button @click="handleLogout" class="logout-btn">Abmelden</button>
      </header>

      <div class="product-grid">
        <div v-for="p in products" :key="p.id" class="product-card">
          <h3>{{ p.name }}</h3>
          <p>Produkt-ID: {{ p.id }}</p>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
/* Modernes & Responsives Design (Anforderung Seite 1) */
.app-container {
  font-family: 'Inter', sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f5f7fa;
  padding: 20px;
}

.card {
  background: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  width: 100%;
  max-width: 400px;
}

h2 { text-align: center; color: #2c3e50; }

input, select {
  width: 100%;
  padding: 12px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 5px;
  box-sizing: border-box;
}

.primary-btn {
  width: 100%;
  padding: 12px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
}

.primary-btn:hover { background-color: #3aa876; }

.switch-link {
  text-align: center;
  color: #3498db;
  cursor: pointer;
  margin-top: 15px;
  text-decoration: underline;
}

/* Produktseite Styles */
.product-page { width: 100%; max-width: 900px; }

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.logout-btn {
  background-color: #e74c3c;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.product-card {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  border-top: 4px solid #42b983;
}

/* Responsivität */
@media (max-width: 600px) {
  .header { flex-direction: column; gap: 10px; }
  .product-grid { grid-template-columns: 1fr; }
}
</style>