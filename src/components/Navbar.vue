<script setup lang="ts">
import { ref, onMounted } from "vue"
import { auth } from "../firebase"
import { onAuthStateChanged, signOut } from "firebase/auth"
import { useRouter } from "vue-router"

const router = useRouter()
const user = ref<any>(null)

// Escuchar si el usuario está logueado o no
onMounted(() => {
  onAuthStateChanged(auth, (firebaseUser) => {
    user.value = firebaseUser
  })
})

const cerrarSesion = async () => {
  try {
    await signOut(auth)
    router.push("/login")
  } catch (error) {
    console.error("Error al cerrar sesión", error)
  }
}
</script>

<template>
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark shadow">
    <div class="container">
      <router-link class="navbar-brand fw-bold text-warning" to="/">
        NOTICIAS MAULE
      </router-link>
      
      <div class="navbar-nav ms-auto d-flex align-items-center">
        <router-link class="nav-link" to="/">Inicio</router-link>

        <template v-if="!user">
          <router-link class="nav-link" to="/login">Entrar</router-link>
          <router-link class="nav-link" to="/register">Registrarse</router-link>
        </template>

        <template v-else>
          <router-link class="nav-link text-info" to="/create-news">Publicar</router-link>
          <span class="nav-link disabled text-light small">| {{ user.email }} |</span>
          <button class="btn btn-outline-danger btn-sm ms-2" @click="cerrarSesion">
            Salir
          </button>
        </template>
      </div>
    </div>
  </nav>
</template>