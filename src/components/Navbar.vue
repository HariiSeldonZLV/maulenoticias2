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
  getPreciseWeather() // Llamamos a la función de precisión
})

const cerrarSesion = async () => {
  try {
    await signOut(auth)
    router.push("/login")
  } catch (error) { console.error(error) }
}

const getPreciseWeather = () => {
  if (navigator.geolocation) {
    // Esto disparará la solicitud de permiso en el navegador
    navigator.geolocation.getCurrentPosition(async (position) => {
      const { latitude, longitude } = position.coords
      try {
        // Le pasamos las coordenadas exactas a wttr.in
        const response = await fetch(`https://wttr.in/${latitude},${longitude}?format=3`)
        if (response.ok) {
          weatherInfo.value = await response.text()
        }
      } catch (error) {
        console.warn("Error al obtener datos del clima")
      }
    }, (error) => {
      console.warn("El usuario denegó la ubicación o hubo un error:", error.message)
      // Si falla o deniega, podemos dejarlo vacío o usar la IP como respaldo
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
        <span class="badge border border-warning text-warning">{{ weatherInfo }}</span>
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
/* Estilo un poco más "Maule" para el clima */
.badge {
  background: transparent;
  font-size: 0.85rem;
}
</style>