<script setup>
import { ref, reactive } from 'vue'

// --- APP ZUSTAND ---
const currentView = ref('login') // Startansicht ist Login
const products = ref([])
const errorMessage = ref('')
const successMessage = ref('')

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

// --- HILFSFUNKTIONEN ---
function clearMessages() {
  errorMessage.value = ''
  successMessage.value = ''
}

// 1. REGISTRIERUNG
async function handleRegister() {
  clearMessages()

  // Frontend-Validierung
  if (!regForm.username.includes('@')) {
    errorMessage.value = "Bitte eine gültige E-Mail-Adresse eingeben."
    return
  }

  if (regForm.password.length < 6) {
    errorMessage.value = "Das Kennwort muss mindestens 6 Zeichen lang sein."
    return
  }

  try {
    const response = await fetch(`${API_BASE}/user/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(regForm)
    })

    if (response.ok) {
      successMessage.value = "Registrierung erfolgreich! Bitte logge dich ein."
      currentView.value = 'login'

      Object.assign(regForm, {
        username: '',
        firstname: '',
        lastname: '',
        password: '',
        language: 'de'
      })

    } else {
      errorMessage.value = "Fehler bei der Registrierung. Benutzer existiert evtl. bereits."
    }

  } catch (error) {
    errorMessage.value = "Netzwerkfehler: Backend nicht erreichbar."
  }
}

// 2. LOGIN
async function handleLogin() {
  clearMessages()

  try {
    const response = await fetch(`${API_BASE}/user/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(loginForm)
    })

    if (response.ok) {
      const data = await response.json()

      if (data && data.token) {
        localStorage.setItem('auth_token', data.token)
      } else {
        localStorage.setItem('auth_token', 'dummy-token')
      }

      loginForm.password = ''

      await fetchProducts()

      currentView.value = 'products'

    } else {
      errorMessage.value = "Anmeldung fehlgeschlagen. E-Mail oder Passwort falsch."
    }

  } catch (error) {
    errorMessage.value = "Login-Fehler. Backend nicht erreichbar."
  }
}

// 3. PRODUKTE LADEN
async function fetchProducts() {
  clearMessages()

  try {
    const token = localStorage.getItem('auth_token')

    const headers = { 'Content-Type': 'application/json' }

    if (token) {
      headers['Authorization'] = `Bearer ${token}`
    }

    const response = await fetch(`${API_BASE}/product/list`, {
      method: 'GET',
      headers: headers
    })

    if (response.ok) {
      const data = await response.json()

      products.value = Array.isArray(data)
          ? data
          : (data.products || data.data || [])

    } else {
      errorMessage.value = "Fehler beim Laden der Produkte."
    }

  } catch (error) {
    errorMessage.value = "Produkte konnten nicht geladen werden. Backend erreichbar?"
  }
}

// 4. LOGOUT
function handleLogout() {
  clearMessages()

  localStorage.removeItem('auth_token')

  products.value = []

  currentView.value = 'login'
}
</script>