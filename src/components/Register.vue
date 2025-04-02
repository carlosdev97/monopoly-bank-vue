<template>
  <div
    class="container d-flex justify-content-center align-items-center flex-column vh-100 w-100"
  >
    <div class="row w-100 d-flex justify-content-center align-items-center">
      <div
        class="col col-md-8 col-lg-6 bg-semi-transparent shadow-lg rounded-4 p-5"
      >
        <h2 class="text-center">Registrarse</h2>
        <form
          class="d-flex flex-column gap-3 mt-4"
          @submit.prevent="handleSubmit"
        >
          <input
            type="text"
            class="form-control mb-3"
            id="inputUsername"
            placeholder="Nombre de usuario"
            v-model="username"
            required
          />
          <input
            type="email"
            class="form-control mb-3"
            id="inputEmail"
            placeholder="Correo electrónico"
            v-model="email"
            required
          />
          <input
            type="password"
            class="form-control mb-3"
            id="inputPassword"
            placeholder="Contraseña"
            v-model="password"
            required
          />
          <input
            type="password"
            class="form-control mb-3"
            id="inputConfirmPassword"
            placeholder="Confirmar contraseña"
            v-model="confirmPassword"
            required
          />
          <button type="submit" class="btn btn-primary w-100">Registrar</button>
        </form>
      </div>
    </div>
    <Toast ref="toastRef" :message="toastMessage" :color="color" />
  </div>
</template>

<script>
import { ref } from "vue";
import {
  auth,
  db,
  createUserWithEmailAndPassword,
  setDoc,
  doc,
} from "../firebaseConfig";
import { query, collection, where, getDocs } from "firebase/firestore";
import { useRouter } from "vue-router";
import Toast from "./Toast.vue";

export default {
  components: {
    Toast,
  },
  setup() {
    const username = ref("");
    const email = ref("");
    const password = ref("");
    const confirmPassword = ref("");
    const toastRef = ref(null);
    const toastMessage = ref("");
    const color = ref(""); // Color del Toast, puedes cambiarlo a "success", "danger", etc.

    // Función para mostrar el Toast con un mensaje específico
    const showNotification = (message, newColor) => {
      toastMessage.value = message;
      color.value = newColor;
      if (toastRef.value) {
        toastRef.value.showToast();
      }
    };

    // Función para verificar si el usuario ya existe
    const checkUserExists = async (email) => {
      const userRef = query(
        collection(db, "users"),
        where("email", "==", email)
      );
      const userSnapshot = await getDocs(userRef);
      return !userSnapshot.empty; // Si el usuario ya existe, devuelve true
    };

    // Función para manejar el registro de usuarios
    const handleSubmit = async () => {
      if (password.value !== confirmPassword.value) {
        showNotification("Las contraseñas no coinciden", "bg-danger");
        return;
      }

      console.log("Verificando usuario...");
      const userExists = await checkUserExists(email.value);
      if (userExists) {
        showNotification("Este correo ya está registrado", "bg-danger");
        return;
      }

      try {
        const userCredential = await createUserWithEmailAndPassword(
          auth,
          email.value,
          password.value
        );
        const user = userCredential.user;

        // Guardar en Firestore
        await setDoc(doc(db, "users", user.uid), {
          username: username.value,
          email: user.email,
          createdAt: new Date(),
        });

        showNotification("Usuario registrado con éxito", "bg-success");
        router.push("/login");
      } catch (error) {
        console.log(error.message);
      }
    };

    return {
      username,
      email,
      password,
      confirmPassword,
      handleSubmit,
      toastRef,
      toastMessage,
      color,
      showNotification,
    };
  },
};
</script>

<style scoped>
.bg-semi-transparent {
  background: rgba(255, 255, 255, 0.8);
}
</style>
