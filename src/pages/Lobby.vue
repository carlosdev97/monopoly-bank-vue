<template>
  <div class="container">
    <div
      class="lobby d-flex justify-content-center align-items-center flex-column vh-100 w-100"
    >
      <h1 class="text-white">¡Hola, {{ username }}!</h1>
      <div class="gap-3 d-flex mt-4">
        <RouterLink to="/create-game" class="btn btn-success">
          Crear partida
        </RouterLink>
        <RouterLink to="/join-game" class="btn btn-primary">
          Unirse a partida
        </RouterLink>
        <button class="btn btn-danger" @click="handleExit">Salir</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from "vue";
import { signOut, onAuthStateChanged } from "firebase/auth";
import { doc, getDoc } from "firebase/firestore";
import { auth, db } from "../firebaseConfig";
import { useRouter, RouterLink } from "vue-router";

export default {
  name: "Lobby",

  setup() {
    const router = useRouter();
    const user = ref(null);
    const username = ref("");
    let loadUser = null;

    // Función para salir del juego (cerrar sesión)
    const handleExit = async () => {
      await signOut(auth);
      router.push("/login");
    };

    // Se ejecuta cuando el componente se monta
    onMounted(() => {
      const loadUser = onAuthStateChanged(auth, async (currentUser) => {
        if (currentUser) {
          user.value = currentUser;

          // Obtener username desde Firestore
          const userDoc = await getDoc(doc(db, "users", currentUser.uid));
          if (userDoc.exists()) {
            username.value = userDoc.data().username;
          }
        } else {
          router.push("/login");
        }
      });
    });

    // Limpiar la suscripción cuando el componente se desmonte
    onUnmounted(() => {
      if (loadUser) loadUser();
    });

    return {
      user,
      username,
      handleExit,
    };
  },
};
</script>

<style scoped>
.lobby {
  text-align: center;
  padding: 20px;
}
</style>
