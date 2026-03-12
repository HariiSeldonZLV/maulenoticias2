<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const clima = ref({ temp: 0, icon: "", ciudad: "Cargando..." });

const irCategoria = (categoria: string) => {
  router.push({
    path: "/",
    query: { categoria }
  });
};

// Función para obtener el clima basado en coordenadas
const obtenerClima = async (lat: number, lon: number, nombreCiudad: string = "Local") => {
  try {
    const res = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true`);
    const data = await res.json();
    clima.value.temp = Math.round(data.current_weather.temperature);
    clima.value.ciudad = nombreCiudad;
    
    const code = data.current_weather.weathercode;
    clima.value.icon = code === 0 ? "☀️" : code < 4 ? "🌤️" : "☁️";
  } catch (error) {
    console.error("Error al obtener clima", error);
  }
};

// Lógica para pedir ubicación
const inicializarClima = () => {
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        // El usuario aceptó: usamos sus coordenadas reales
        obtenerClima(position.coords.latitude, position.coords.longitude, "Tu ubicación");
      },
      () => {
        // El usuario rechazó o hubo error: usamos Talca por defecto
        obtenerClima(-35.4264, -71.6554, "Talca");
      }
    );
  } else {
    // El navegador no soporta geolocalización
    obtenerClima(-35.4264, -71.6554, "Talca");
  }
};

onMounted(() => {
  inicializarClima();
});
</script>

<template>
<nav class="navbar navbar-expand-lg navbar-dark bg-black border-bottom border-danger">
  <div class="container">

    <router-link class="navbar-brand" to="/">Noticias Maule</router-link>

    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navbarNav">

      <ul class="navbar-nav me-auto">
        <li class="nav-item"><a class="nav-link" style="cursor: pointer" @click="irCategoria('todas')">Todas</a></li>
        <li class="nav-item"><a class="nav-link" style="cursor: pointer" @click="irCategoria('politica')">Política</a></li>
        <li class="nav-item"><a class="nav-link" style="cursor: pointer" @click="irCategoria('deportes')">Deportes</a></li>
        <li class="nav-item"><a class="nav-link" style="cursor: pointer" @click="irCategoria('tecnologia')">Tecnología</a></li>
        <li class="nav-item"><a class="nav-link" style="cursor: pointer" @click="irCategoria('economia')">Economía</a></li>

        <li class="nav-item">
          <a class="nav-link" href="https://HariiSeldonZLV.github.io/bursatil/" target="_blank" rel="noopener noreferrer">
            Bursátil
          </a>
        </li>
      </ul>

      <div class="d-flex align-items-center">
        <div v-if="clima.temp" class="text-white me-3 d-none d-md-block">
          <span class="text-secondary small">{{ clima.ciudad }}:</span> 
          <strong>{{ clima.icon }} {{ clima.temp }}°C</strong>
        </div>

        <router-link to="/create-news" class="btn btn-warning me-2">Publicar</router-link>
        <router-link to="/login" class="btn btn-outline-light me-2">Login</router-link>
        <router-link to="/register" class="btn btn-primary">Registro</router-link>
      </div>

    </div>
  </div>
</nav>
</template>