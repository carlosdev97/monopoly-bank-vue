<template>
  <div
    class="container d-flex justify-content-center align-items-center flex-column vh-100 w-100"
  >
    <div class="row">
      <div
        class="col d-flex flex-column justify-content-center align-items-center gap-3"
      >
        <h1>¡Bienvenido a Monopoly Bank!</h1>
        <p class="fs-5">Por favor elija una opción</p>
        <div class="d-flex gap-3">
          <RouterLink
            to="/register"
            class="btn btn-primary mt-3 px-5 rounded-5 fs-5"
          >
            Registrarse
          </RouterLink>
          <RouterLink
            to="/login"
            class="btn btn-secondary mt-3 px-5 rounded-5 fs-5"
          >
            Iniciar sesión
          </RouterLink>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { RouterLink } from "vue-router";
</script>
