<template>
  <div class="container mt-5">
    <div class="row justify-content-center">
      <div class="col-md-5">
        <div class="card p-4 shadow bg-dark text-white border-primary">
          <h2 class="mb-4">Registro</h2>
          
          <button class="btn btn-outline-light w-100 mb-4 d-flex align-items-center justify-content-center" @click="registroGoogle">
  <img src="https://fonts.gstatic.com/s/i/productlogos/googleg/v6/24px.svg" width="20" class="me-2" alt="Google Logo">
  Registrarse con Google
</button>

          <div class="text-center mb-4"><hr> o usa tu email <hr></div>

          <div class="mb-3">
            <label class="form-label">Email</label>
            <input v-model="email" type="email" class="form-control" />
          </div>
          <div class="mb-3">
            <label class="form-label">Contraseña</label>
            <input v-model="password" type="password" class="form-control" />
          </div>

          <button class="btn btn-primary w-100" @click="register">Crear Cuenta</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue"
import { auth, db } from "../firebase"
import { createUserWithEmailAndPassword, GoogleAuthProvider, signInWithPopup } from "firebase/auth"
import { doc, setDoc } from "firebase/firestore"
import { useRouter } from "vue-router"

const email = ref("")
const password = ref("")
const router = useRouter()

// Registro con Google
const registroGoogle = async () => {
  try {
    const provider = new GoogleAuthProvider()
    const result = await signInWithPopup(auth, provider)
    await setDoc(doc(db, "users", result.user.uid), {
      uid: result.user.uid,
      email: result.user.email,
      createdAt: new Date()
    })
    router.push("/")
  } catch (error) {
    console.error(error)
  }
}

// Registro con Email (Tu función actual)
const register = async () => {
  try {
    const userCredential = await createUserWithEmailAndPassword(auth, email.value.trim(), password.value)
    await setDoc(doc(db, "users", userCredential.user.uid), {
      uid: userCredential.user.uid,
      email: userCredential.user.email,
      createdAt: new Date()
    })
    router.push("/login")
  } catch (error) {
    console.error(error)
  }
}
</script>

<style scoped>
/* Estilo para el botón de Google */
.btn-outline-light {
  border-color: rgba(255, 255, 255, 0.2);
  font-weight: 500;
  background-color: #fff;
  color: #000; /* Texto negro para que resalte el logo */
}

.btn-outline-light:hover {
  background-color: #f1f1f1;
  color: #000;
  border-color: #fff;
}

/* El divisor "o usa tu email" */
hr {
  display: inline-block;
  width: 30%;
  vertical-align: middle;
  border-top: 1px solid rgba(255, 255, 255, 0.3);
}
</style>