<template>
  <div
    class="container d-flex justify-content-center align-items-center flex-column vh-100 w-100"
  >
    <div class="row w-100 d-flex justify-content-center align-items-center">
      <div
        class="col col-md-8 col-lg-6 bg-semi-transparent shadow-lg rounded-4 p-5"
      >
        <h3
          class="text-center d-flex align-items-center justify-content-center"
        >
          Código:{{ " " }} <span> {{ codeGame }}</span>
          <button class="btn btn-success mx-2" @click="copyCode">Copiar</button>
        </h3>

        <h5 class="text-center">
          Estado de la partida: <span class="badge bg-info">{{ state }}</span>
        </h5>

        <h4 class="text-center mt-4">Lista de participantes</h4>
        <ul class="list-group">
          <li
            v-for="(p, index) in participants"
            :key="index"
            class="list-group-item"
          >
            {{ p.name }} - {{ p.balance }}
          </li>
        </ul>

        <div v-if="state === 'waiting'" class="d-grid">
          <button class="btn btn-primary mt-3" @click="startGame">
            Iniciar partida
          </button>
          <button class="btn btn-danger mt-3">Volver</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from "vue";
import { toast } from "vue3-toastify";
import { useRouter } from "vue-router";
import { doc, setDoc, onSnapshot, updateDoc } from "firebase/firestore";
import { auth, db } from "../firebaseConfig";

export default {
  name: "NewGame",
  setup() {
    const router = useRouter();
    const codeGame = ref("");
    const participants = ref([]);
    const state = ref("En espera");
    const isExecuted = ref(false);

    const copyCode = () => {
      navigator.clipboard.writeText(codeGame);
      toast.success("¡Código copiado al portapapeles!");
    };

    const startGame = async () => {
      const gameRef = doc(db, "games", codeGame.value);
      await updateDoc(gameRef, {
        state: "Jugando",
      });
    };

    onMounted(() => {
      if (isExecuted.value) return;
      isExecuted.value = true;

      const createNewGame = async () => {
        const user = auth.currentUser;
        if (!user) {
          toast.error("Debes iniciar sesión para crear un juego");
          router.push("/login");
          return;
        }

        const username = user.displayName || user.email.split("@")[0];
        const gameCode = Math.floor(1000 + Math.random() * 9000).toString();

        const gameRef = doc(db, "games", gameCode);

        await setDoc(gameRef, {
          code: gameCode,
          state: "En espera",
          host: user.uid,
          hostName: username,
          participants: [
            {
              uid: user.uid,
              name: username,
              balance: 1500,
            },
          ],
        });

        codeGame.value = gameCode;
        participants.value = [
          {
            uid: user.uid,
            name: username,
            balance: 1500,
          },
        ];
        state.value = "En espera";

        const unsub = onSnapshot(gameRef, (docSnap) => {
          if (docSnap.exists()) {
            participants.value = docSnap.data().participants;
            state.value = docSnap.data().state;
            if (docSnap.data().state === "Jugando") {
              router.push(`/game/${gameCode}`);
            }
          }
        });

        onUnmounted(() => {
          unsub();
        });
      };

      createNewGame();
    });

    return {
      copyCode,
      codeGame,
      participants,
      state,
      startGame,
    };
  },
};
</script>

<style scoped>
.bg-semi-transparent {
  background-color: rgba(255, 255, 255, 0.5);
}
.new-game {
  padding: 20px;
  text-align: center;
}
</style>
