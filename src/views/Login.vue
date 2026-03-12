<script setup lang="ts">
import { ref } from "vue"
import { auth } from "../firebase" // Asegúrate de que firebase.js exporte 'auth'
import { signInWithEmailAndPassword } from "firebase/auth"
import { useRouter } from "vue-router"

const router = useRouter()
const email = ref("")
const password = ref("")
const errorMsg = ref("")

const iniciarSesion = async () => {
  try {
    // Usamos .trim() para evitar el error de email inválido por espacios
    await signInWithEmailAndPassword(auth, email.value.trim(), password.value)
    router.push("/")
  } catch (error: any) {
    console.error("Error de login:", error.code)
    errorMsg.value = "Correo o contraseña incorrectos"
  }
}
</script>

<template>
  <div class="container mt-5">
    <div class="row justify-content-center">
      <div class="col-md-5">
        <div class="card p-4 shadow bg-dark text-white border-danger">
          <h2 class="mb-4">Iniciar Sesión</h2>
          
          <div v-if="errorMsg" class="alert alert-danger">{{ errorMsg }}</div>

          <div class="mb-3">
            <label class="form-label">Email</label>
            <input v-model="email" type="email" class="form-control" placeholder="tu@email.com" />
          </div>

          <div class="mb-3">
            <label class="form-label">Contraseña</label>
            <input v-model="password" type="password" class="form-control" />
          </div>

          <button class="btn btn-primary w-100" @click="iniciarSesion">Entrar</button>
        </div>
      </div>
    </div>
  </div>
</template>