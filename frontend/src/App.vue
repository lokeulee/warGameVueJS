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
        <img :src="player.image" />
      </div>
      <button @click="startGame()">Start Game!</button>
    </div>
    <div v-if="!gameOver" id="game">
      <div v-for="player in playerArr" :key="player.id">
        <h4>{{ player.name }}</h4>
        <div>
          <div class="card">
            <img :src="player.image" />
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

    const suitScore = new Map<string, number>([
      ["DIAMONDS", 0],
      ["CLUBS", 1],
      ["HEARTS", 2],
      ["SPADES", 3],
    ]);

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
            points: 0,
            value: 0,
            suit: "",
            image: "",
            // card: {
            //   code: "",
            //   images: "",
            //   value: 0,
            //   suit: "",
            // },
          };
        }
      }
      constructArr();
    });

    function convertPoints(input: string): number {
      switch (input) {
        case "JACK":
          return 11;
        case "QUEEN":
          return 12;
        case "KING":
          return 13;
        case "ACE":
          return 14;
        default:
          break;
      }
      return parseInt(input);
    }

    function calculateScore(): number {
      let currentHighest;
      let playerID = playerArr.value[0].id;
      currentHighest = playerArr.value[0].value;
      for (let count = 1; count < playerNum.value; count++) {
        if (currentHighest < playerArr.value[count].value) {
          currentHighest = playerArr.value[count].value;
          playerID = playerArr.value[count].id;
        }
        if (currentHighest === playerArr.value[count].value) {
          let comp1 = suitScore.get(playerArr.value[playerID].suit);
          let comp2 = suitScore.get(playerArr.value[count].suit);
          if (comp1 && comp2 && comp1 < comp2) {
            console.log("CHANGEEEE");
            currentHighest = comp2;
          }
        }
      }
      return playerID;
    }

    async function startGame() {
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
        // playerArr.value[count].card = cards[count];
        playerArr.value[count].suit = cards[count].suit;
        playerArr.value[count].value = convertPoints(cards[count].value);
        playerArr.value[count].image = cards[count].image;
        console.log("Card ", cards[count]);
        console.log("VALUEEEEEE", playerArr.value[count].value);
      }
      for (let count = 0; count < playerNum.value; count++) {
        console.log("PRINTING MY STUFF");
        console.log(playerArr.value[count].suit);
        console.log(playerArr.value[count].value);
        console.log(playerArr.value[count].image);
      }

      const roundWinner = calculateScore();
      playerArr.value[roundWinner].points++;
      console.log("ROUNDWINNER", roundWinner);
      console.log("remaining: " + remaining);
      console.log("data getCards: ", data);

      if (remaining < playerNum.value) {
        gameOver.value = true;
        while (playerArr.value.length) {
          playerArr.value.pop();
        }
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
.flex-container {
  display: flex;
  flex-direction: row;
  align-items: center;
}
</style>
