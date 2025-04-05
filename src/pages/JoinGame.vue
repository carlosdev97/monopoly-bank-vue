<template>
  <div
    class="container d-flex justify-content-center align-items-center flex-column vh-100 w-100"
  >
    <div class="row w-100 d-flex justify-content-center align-items-center">
      <div
        class="col col-md-8 col-lg-6 bg-semi-transparent shadow-lg rounded-4 p-5"
      >
        <h1 class="text-center">Unirse a juego</h1>
        <form @submit.prevent="joinGame">
          <input
            type="text"
            class="form-control mb-3 mt-3"
            id="inputCode"
            placeholder="Código"
            v-model="codeGame"
            required
          />
          <button class="btn btn-success w-100" type="submit">Unirse</button>
        </form>
        <span>{{ message }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch } from "vue";
import { toast } from "vue3-toastify";
import {
  getFirestore,
  doc,
  setDoc,
  getDoc,
  onSnapshot,
  updateDoc,
} from "firebase/firestore";
import { auth, db } from "../firebaseConfig";
import { useRouter } from "vue-router";

export default {
  name: "JoinGame",
  setup() {
    const codeGame = ref("");
    const participants = ref([]);
    const router = useRouter();
    const message = ref("");
    const startGame = ref(false);

    const joinGame = async () => {
      if (!codeGame.value) {
        toast.error("Por favor, ingresa un código válido.");
        return;
      }

      const user = auth.currentUser;

      if (!user) {
        toast.error("Debes iniciar sesión para unirte a un juego.");
        router.push("/login");
        return;
      }

      const { displayName, uid } = user;

      const currentPlayer = displayName || "Player " + uid.slice(0, 5);

      console.log("Here!!!");

      const gameRef = doc(db, "games", codeGame.value);

      const gameSnapshot = await getDoc(gameRef);

      if (gameSnapshot.exists()) {
        const gameData = gameSnapshot.data();

        const playerExists = gameData.participants.some(
          (player) => player.uid === uid
        );

        if (playerExists) {
          toast.error("Ya estás en esta partida.");
          return;
        }

        const newPlayer = {
          id: uid,
          name: currentPlayer,
          balance: 1500,
        };

        await updateDoc(gameRef, {
          participants: [...gameData.participants, newPlayer],
        });

        toast.success("Te has unido a la partida con éxito.");
        message.value =
          "Te has unido a la partida con éxito. Esperando que inicie el juego...";

        onSnapshot(gameRef, (docSnapshot) => {
          if (docSnapshot.exists()) {
            const data = docSnapshot.data();
            participants.value = data.participants;

            if (data.state === "Jugando") {
              startGame.value = true;
            }
          }
        });
      } else {
        toast.error("La partida no existe");
      }
    };

    watch(startGame, (newValue) => {
      if (newValue) {
        router.push(`/game/${codeGame.value}`);
      }
    });

    return {
      codeGame,
      participants,
      message,
      startGame,
      joinGame,
    };
  },
};
</script>

<style scoped>
.bg-semi-transparent {
  background-color: rgba(255, 255, 255, 0.5);
}
</style>
