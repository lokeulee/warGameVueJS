<template>
  <div>
    <img alt="Vue logo" src="./assets/logo.png" />
    <h1>War Game!</h1>
    <div id="app" v-if="gameOver">
      <label for="player_count">Number of players:</label>
      <select id="player_count" name="player_count" v-model="playerNum">
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
        <option value="6">6</option>
      </select>
      <div v-for="player in playerArr" :key="player.id">
        <label for="{{player.name}}">{{
          "Player " + (player.id + 1) + " name: "
        }}</label>
        <input type="text" v-model="player.name" /><br /><br />
        <img :src="player.card?.images?.png" />
      </div>
      <button @click="startGame()">Start Game!</button>
    </div>
    <div v-if="!gameOver" id="game">
      <div v-for="player in playerArr" :key="player.id">
        <h4>{{ player.name }}</h4>
        <div>
          <div class="card">
            <img :src="player.card?.images?.png" />
            <!-- <img :src="cardOne?.images?.png" /> -->
          </div>
        </div>
      </div>
      <div>
        <button @click="getCards()">Draw Cards</button>
      </div>
      <div>
        <button @click="restartGame()">Restart Game</button>
      </div>
    </div>
    You selected playernums: <b>{{ playerNum }}</b>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watchEffect } from "vue";
import axios from "axios";
import { Player } from "./types/PlayerType";

export default defineComponent({
  name: "App",
  setup() {
    let playerNum = ref(2);
    let gameOver = ref(true),
      deckID = ref(null);
    let playerArr = ref<Player[]>([]);

    watchEffect(() => {
      console.log("WATCHER CALLED");
      function constructArr() {
        while (playerArr.value.length) {
          playerArr.value.pop();
        }
        for (let count = 0; count <= playerNum.value - 1; count++) {
          playerArr.value[count] = {
            id: count,
            name: "",
            suite: "",
            points: 0,
            card: {},
          };
        }
      }
      constructArr();
    });

    console.log(playerArr.value);

    function startGame() {
      getDeck();
    }

    async function restartGame() {
      gameOver.value = true;
      const { data } = await axios.get(
        "https://www.deckofcardsapi.com/api/deck/new/"
      );

      console.log(data.deck_id);
      axios.get(
        "https://www.deckofcardsapi.com/api/deck/" + data.deck_id + "/shuffle/"
      );
      deckID.value = data.deck_id;
      console.log("Restart Game called");
      while (playerArr.value.length) {
        playerArr.value.pop();
      }
    }

    async function getDeck() {
      gameOver.value = false;
      if (deckID.value == null) {
        const { data } = await axios.get(
          "https://www.deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1"
        );
        deckID.value = data.deck_id;
        console.log("data getDeck", data);
        console.log(deckID);
      }
    }

    async function getCards() {
      const { data } = await axios.get(
        "https://www.deckofcardsapi.com/api/deck/" +
          deckID.value +
          "/draw/?count=" +
          playerNum.value
      );
      const remaining = data.remaining;
      const { cards } = data;
      console.log(cards);
      for (let count = 0; count < playerNum.value; count++) {
        playerArr.value[count].card = cards[count];
        playerArr.value[count].suite = cards[count];
        console.log("Card ", cards[count]);
      }
      for (let count = 0; count < playerNum.value; count++) {
        console.log(playerArr.value[count].card);
      }

      console.log("remaining: " + remaining);
      console.log("data getCards: ", data);

      if (remaining < playerNum.value) {
        gameOver.value = true;
      }
    }

    // getDeck();
    return {
      getDeck,
      playerNum,
      playerArr,
      getCards,
      gameOver,
      restartGame,
      startGame,
    };
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
