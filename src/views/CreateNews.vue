<script setup lang="ts">
import { ref, onMounted } from "vue"
import { db, auth } from "../firebase" // Importamos auth y db desde tu archivo central
import { collection, addDoc, Timestamp } from "firebase/firestore"
import { useRouter } from "vue-router"

const router = useRouter()

const title = ref("")
const content = ref("")
const image = ref("")
const categoria = ref("politica")

// 1. Protección de ruta al montar el componente
onMounted(() => {
  if (!auth.currentUser) {
    alert("Debes iniciar sesión para publicar noticias.");
    router.push("/login");
  }
});

const publicarNoticia = async () => {
  // Verificación extra de seguridad antes de enviar
  if (!title.value || !content.value) {
    alert("Por favor, completa el título y el contenido.");
    return;
  }

  try {
    await addDoc(collection(db, "news"), {
      title: title.value,
      content: content.value,
      image: image.value,
      categoria: categoria.value,
      createdAt: Timestamp.now(),
      authorEmail: auth.currentUser?.email // Es bueno saber quién publicó
    });

    // Limpiar campos
    title.value = ""
    content.value = ""
    image.value = ""
    
    alert("¡Noticia publicada con éxito!");
    router.push("/");
  } catch (error) {
    console.error("Error al publicar:", error);
    alert("No tienes permisos para publicar. Asegúrate de haber iniciado sesión.");
  }
}
</script>

<template>
  <div class="container mt-5">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <div class="card p-4 shadow bg-dark text-white border-warning">
          <h2 class="mb-4 text-warning">Publicar noticia</h2>

          <div class="mb-3">
            <label class="form-label">Título</label>
            <input v-model="title" type="text" class="form-control" placeholder="Escribe un título impactante" />
          </div>

          <div class="mb-3">
            <label class="form-label">Categoría</label>
            <select v-model="categoria" class="form-control">
              <option value="politica">Política</option>
              <option value="deportes">Deportes</option>
              <option value="tecnologia">Tecnología</option>
              <option value="economia">Economía</option>
            </select>
          </div>

          <div class="mb-3">
            <label class="form-label">Contenido</label>
            <textarea v-model="content" rows="6" class="form-control" placeholder="Desarrolla la noticia aquí..."></textarea>
          </div>

          <div class="mb-3">
            <label class="form-label">URL de imagen</label>
            <input v-model="image" type="text" class="form-control" placeholder="https://ejemplo.com/imagen.jpg" />
          </div>

          <button class="btn btn-warning w-100 fw-bold" @click="publicarNoticia">
            🚀 Enviar Noticia
          </button>
        </div>
      </div>
    </div>
  </div>
</template>