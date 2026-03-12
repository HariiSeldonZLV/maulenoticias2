<script setup lang="ts">
import { ref, onMounted } from "vue"
import { auth } from "../firebase"
import { onAuthStateChanged, signOut } from "firebase/auth"
import { useRouter } from "vue-router"

const router = useRouter()
const user = ref<any>(null)

// Variables para el clima
const weather = ref<any>(null)
const city = ref("Cargando clima...")

// Lógica de Autenticación
onMounted(() => {
  onAuthStateChanged(auth, (firebaseUser) => {
    user.value = firebaseUser
  })
  getWeather() // Llamamos al clima al montar
})

const cerrarSesion = async () => {
  try {
    await signOut(auth)
    router.push("/login")
  } catch (error) {
    console.error("Error al cerrar sesión", error)
  }
}

// Lógica del Clima
const getWeather = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(async (position) => {
      const { latitude, longitude } = position.coords
      try {
        const API_KEY = "tu_api_key_aqui" // Reemplaza con tu clave de OpenWeather
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${API_KEY}&units=metric&lang=es`
        )
        const data = await response.json()
        weather.value = data
        city.value = data.name
      } catch (error) {
        console.error("Error obteniendo el clima", error)
        city.value = "Clima no disponible"
      }
    })
  }
}
</script>

<template>
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark shadow">
    <div class="container">
      <router-link class="navbar-brand fw-bold text-warning" to="/">
        NOTICIAS MAULE
      </router-link>

      <div v-if="weather" class="text-white small d-none d-md-flex align-items-center me-4">
        <img :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}.png`" width="30" />
        <span>{{ city }}: {{ Math.round(weather.main.temp) }}°C</span>
      </div>

      <div class="navbar-nav ms-auto d-flex align-items-center">
        <router-link class="nav-link" to="/">Inicio</router-link>

        <template v-if="!user">
          <router-link class="nav-link" to="/login">Entrar</router-link>
          <router-link class="nav-link" to="/register">Registrarse</router-link>
        </template>

        <template v-else>
          <router-link class="nav-link text-info fw-bold" to="/create-news">Publicar</router-link>
          <span class="nav-link disabled text-light small d-none d-sm-inline">| {{ user.email }} |</span>
          <button class="btn btn-outline-danger btn-sm ms-2" @click="cerrarSesion">
            Salir
          </button>
        </template>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.navbar {
  border-bottom: 2px solid #ffc107;
}
.nav-link:hover {
  color: #ffc107 !important;
}
</style>