<script setup lang="ts">
import { ref, onMounted } from "vue"
import { auth } from "../firebase"
import { onAuthStateChanged, signOut } from "firebase/auth"
import { useRouter } from "vue-router"

const router = useRouter()
const user = ref<any>(null)
const weatherInfo = ref("")

onMounted(() => {
  onAuthStateChanged(auth, (firebaseUser) => {
    user.value = firebaseUser
  })
  getPreciseWeather()
})

const cerrarSesion = async () => {
  try {
    await signOut(auth)
    router.push("/login")
  } catch (error) { console.error(error) }
}

const getPreciseWeather = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(async (position) => {
      const { latitude, longitude } = position.coords
      try {
        // PASO 1: Traducir coordenadas a nombre de ciudad usando Nominatim (Gratis)
        const geoResponse = await fetch(`https://nominatim.openstreetmap.org/reverse?lat=${latitude}&lon=${longitude}&format=json`)
        const geoData = await geoResponse.json()
        
        // Sacamos la ciudad, pueblo o villa
        const locationName = geoData.address.city || geoData.address.town || geoData.address.village || "Maule"

        // PASO 2: Pedir el clima para ese nombre específico
        const weatherResponse = await fetch(`https://wttr.in/${locationName}?format=%l:+%c+%t`)
        if (weatherResponse.ok) {
          weatherInfo.value = await weatherResponse.text()
        }
      } catch (error) {
        console.warn("Error en el clima:", error)
      }
    }, (error) => {
      // Si el usuario no da permiso, usamos la IP como respaldo
      fetch('https://wttr.in/?format=3')
        .then(res => res.text())
        .then(data => weatherInfo.value = data)
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

      <div v-if="weatherInfo" class="text-white small d-none d-md-flex align-items-center me-4">
        <span class="badge border border-warning text-warning fw-normal px-3 py-2">
          {{ weatherInfo }}
        </span>
      </div>

      <div class="navbar-nav ms-auto d-flex align-items-center">
        <router-link class="nav-link" to="/">Inicio</router-link>

        <template v-if="!user">
          <router-link class="nav-link" to="/login">Entrar</router-link>
          <router-link class="nav-link" to="/register">Registrarse</router-link>
        </template>

        <template v-else>
          <router-link class="nav-link text-info fw-bold" to="/create-news">Publicar</router-link>
          <button class="btn btn-outline-danger btn-sm ms-2" @click="cerrarSesion">
            Salir
          </button>
        </template>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.navbar { border-bottom: 2px solid #ffc107; }
.badge {
  background: rgba(255, 193, 7, 0.1);
  letter-spacing: 0.5px;
  border-radius: 20px;
}
</style>