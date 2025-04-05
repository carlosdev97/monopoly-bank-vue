<template>
  <div
    class="container d-flex justify-content-center align-items-center flex-column vh-100 w-100"
  >
    <div class="row w-100 d-flex justify-content-center align-items-center">
      <div
        class="col col-md-8 col-lg-6 bg-semi-transparent shadow-lg rounded-4 p-5"
      >
        <h2 class="text-center">Iniciar sesión</h2>
        <form
          @submit.prevent="handleSubmit"
          class="d-flex flex-column gap-3 mt-4"
        >
          <input
            v-model="email"
            type="email"
            class="form-control mb-3"
            id="inputEmail"
            placeholder="Correo electrónico"
            required
          />
          <input
            v-model="password"
            type="password"
            class="form-control mb-3"
            id="inputPassword"
            placeholder="Contraseña"
            required
          />
          <button type="submit" class="btn btn-primary w-100">Ingresar</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import { toast } from "vue3-toastify";
import { signInWithEmailAndPassword } from "firebase/auth";
import { auth } from "../firebaseConfig";
import { useRouter } from "vue-router";

export default {
  setup() {
    const email = ref("");
    const password = ref("");

    const router = useRouter();

    const handleSubmit = async () => {
      if (!email.value || !password.value) {
        toast.error("Por favor, completa todos los campos.");
        return;
      }

      try {
        await signInWithEmailAndPassword(auth, email.value, password.value);
        toast.success("Inicio de sesión exitoso.");
        router.push("/lobby"); // Redirigir a la página de lobby después de iniciar sesión
      } catch (error) {
        toast.error("Error al iniciar sesión. Inténtalo de nuevo.");
      }
    };

    return {
      email,
      password,
      handleSubmit,
    };
  },
};
</script>

<style scoped>
.bg-semi-transparent {
  background-color: rgba(255, 255, 255, 0.5);
}
</style>
