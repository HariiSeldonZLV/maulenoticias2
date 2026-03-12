<script setup lang="ts">
import { ref } from "vue"
import { createUserWithEmailAndPassword } from "firebase/auth"
import { auth, db } from "../firebase"
import { doc, setDoc } from "firebase/firestore"
import { useRouter } from "vue-router"

const email = ref("")
const password = ref("")
const errorMsg = ref("")
const router = useRouter()

const register = async () => {
  try {
    // Usamos .trim() para limpiar espacios accidentales
    const userCredential = await createUserWithEmailAndPassword(
      auth,
      email.value.trim(),
      password.value
    )

    const user = userCredential.user

    // Guardar perfil adicional en Firestore
    await setDoc(doc(db, "users", user.uid), {
      uid: user.uid,
      email: user.email,
      role: 'user', // Puedes añadir roles aquí
      createdAt: new Date()
    })

    alert("¡Cuenta creada con éxito!")
    router.push("/login")

  } catch (error: any) {
    console.error(error.code)
    if (error.code === 'auth/email-already-in-use') {
      errorMsg.value = "Este correo ya está registrado."
    } else if (error.code === 'auth/weak-password') {
      errorMsg.value = "La contraseña debe tener al menos 6 caracteres."
    } else {
      errorMsg.value = "Error al registrar: " + error.message
    }
  }
}
</script>

<template>
  <div class="container mt-5">
    <div class="row justify-content-center">
      <div class="col-md-5">
        <div class="card p-4 shadow bg-dark text-white border-primary">
          <h2 class="mb-4">Crear Cuenta</h2>
          
          <div v-if="errorMsg" class="alert alert-danger">{{ errorMsg }}</div>

          <div class="mb-3">
            <label class="form-label">Correo Electrónico</label>
            <input 
              v-model="email" 
              type="email" 
              class="form-control" 
              placeholder="nombre@ejemplo.com"
            />
          </div>

          <div class="mb-3">
            <label class="form-label">Contraseña</label>
            <input 
              v-model="password" 
              type="password" 
              class="form-control" 
              placeholder="Mínimo 6 caracteres"
            />
          </div>

          <button class="btn btn-primary w-100 mt-3" @click="register">
            Registrarse
          </button>

          <p class="mt-3 text-center">
            ¿Ya tienes cuenta? <router-link to="/login" class="text-warning">Inicia sesión</router-link>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>