<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const isLoginMode = ref(true) // Switch zwischen Login und Register
const API_URL = "https://ipt71.kuno-schuerch.bbzwinf.ch"

// Daten-Objekt für die Registrierung
const regData = reactive({
  username: '', firstname: '', lastname: '', password: '', language: 'de'
})

// Daten-Objekt für den Login
const loginData = reactive({
  username: '', password: ''
})

// Funktion: Registrieren
async function handleRegister() {
  // Einfache Validierung
  if (regData.password.length < 5) return alert("Passwort zu kurz")

  try {
    const res = await fetch(`${API_URL}/user/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(regData)
    })

    if (res.ok) {
      alert("Erfolgreich! Bitte logge dich nun ein.")
      isLoginMode.value = true
    } else {
      alert("Fehler bei der Registrierung")
    }
  } catch (err) { alert("Serverfehler") }
}

// Funktion: Login
async function handleLogin() {
  try {
    const res = await fetch(`${API_URL}/user/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(loginData)
    })

    if (res.ok) {
      localStorage.setItem('isLoggedIn', 'true')
      router.push('/products')
    } else {
      alert("Login-Daten falsch")
    }
  } catch (err) { alert("Serverfehler") }
}
</script>

<template>
  <div class="auth-container">
    <!-- REGISTRIERUNG -->
    <div v-if="!isLoginMode" class="card">
      <h1>Registrieren</h1>
      <form @submit.prevent="handleRegister">
        <input v-model="regData.username" type="email" placeholder="E-Mail" required />
        <input v-model="regData.firstname" type="text" placeholder="Vorname" required />
        <input v-model="regData.lastname" type="text" placeholder="Nachname" required />
        <input v-model="regData.password" type="password" placeholder="Passwort" required />
        <select v-model="regData.language">
          <option value="de">Deutsch</option>
          <option value="en">Englisch</option>
          <option value="fr">Französisch</option>
        </select>
        <button type="submit">Konto erstellen</button>
      </form>
      <p @click="isLoginMode = true">Schon ein Konto? Login</p>
    </div>

    <!-- LOGIN -->
    <div v-else class="card">
      <h1>Login</h1>
      <form @submit.prevent="handleLogin">
        <input v-model="loginData.username" type="email" placeholder="E-Mail" required />
        <input v-model="loginData.password" type="password" placeholder="Passwort" required />
        <button type="submit">Einloggen</button>
      </form>
      <p @click="isLoginMode = false">Neu hier? Registrieren</p>
    </div>
  </div>
</template>

<style scoped>
.auth-container { display: flex; justify-content: center; padding-top: 50px; }
.card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); width: 300px; }
input, select, button { width: 100%; margin-bottom: 10px; padding: 8px; box-sizing: border-box; }
button { background: #42b983; color: white; border: none; cursor: pointer; }
p { font-size: 0.8rem; cursor: pointer; color: blue; text-align: center; }
</style>