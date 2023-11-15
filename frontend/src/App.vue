<template>
  <div>
    <h1>War Game!</h1>
    <div v-if="gameOverScreen" style="font-size: 50px">
      <div>
        The winner is:
        <b>{{
          playerArr[winnerID].name
            ? playerArr[winnerID].name + "!!"
            : "Player " + playerArr[winnerID].id
        }}</b>
      </div>
      <button @click="restartGame()">Restart!</button>
    </div>

    <div v-if="!gameOverScreen">
      <div id="app" v-if="gameOver">
        <label for="player_count">Number of players:</label>
        <select id="player_count" name="player_count" v-model="playerNum">
          <option value="2">2</option>
          <option value="3">3</option>
          <option value="4">4</option>
          Í
          <option value="5">5</option>
          <option value="6">6</option>
        </select>
        <div v-for="player in playerArr" :key="player.id">
          <div>
            <label for="{{player.name}}">{{
              "Player " + (player.id + 1) + " name: "
            }}</label>
            <input type="text" v-model="player.name" /><br /><br />
            <img :src="player.image" />
          </div>
        </div>

        <button @click="startGame()">Start Game!</button>
      </div>
      <div v-if="!gameOver" id="game">
        <div
          style="display: flex; flex-direction: row; justify-content: center"
        >
          <div
            v-for="player in playerArr"
            :key="player.id"
            style="margin: 15px; padding: 10px"
          >
            <div
              v-if="player.image"
              style="
                display: flex;
                flex-direction: row;
                justify-content: center;
              "
            >
              <h4 style="margin-right: 10px">
                {{ player.name ? player.name : "Player " + (player.id + 1) }}
              </h4>
              <h4>{{ "Score: " + player.points }}</h4>
            </div>

            <div
              style="
                display: flex;
                flex-direction: row;
                justify-content: center;
              "
            >
              <img :src="player.image" style="width: 100px" />
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
    </div>
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
    let gameOverScreen = ref(false);
    let gameOver = ref(true);
    let deckID = ref(null);
    let playerArr = ref<Player[]>([]);
    let winnerID = ref(0);

    const suitScore = new Map<string, number>([
      ["DIAMONDS", 0],
      ["CLUBS", 1],
      ["HEARTS", 2],
      ["SPADES", 3],
    ]);

    watchEffect(() => {
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
            currentHighest = comp2;
          }
        }
      }
      return playerID;
    }

    function findWinnerID(): void {
      let mostPoints = 0;
      for (let count = 0; count < playerNum.value; count++) {
        if (mostPoints < playerArr.value[count].points) {
          mostPoints = playerArr.value[count].points;
          winnerID.value = playerArr.value[count].id;
        }
      }
    }

    async function getCards() {
      const button = document.querySelector("button");

      if (button) {
        button.disabled = true;
      }

      const { data } = await axios.get(
        "https://www.deckofcardsapi.com/api/deck/" +
          deckID.value +
          "/draw/?count=" +
          playerNum.value
      );
      const remaining = data.remaining;

      const { cards } = data;
      for (let count = 0; count < playerNum.value; count++) {
        playerArr.value[count].suit = cards[count].suit;
        playerArr.value[count].value = convertPoints(cards[count].value);
        playerArr.value[count].image = cards[count].image;
      }

      const roundWinner = calculateScore();
      playerArr.value[roundWinner].points++;

      if (remaining < playerNum.value) {
        gameOver.value = true;
        gameOverScreen.value = true;
        findWinnerID();
      }
      if (button) {
        button.disabled = false;
      }
    }

    async function restartGame() {
      gameOver.value = true;
      gameOverScreen.value = false;

      while (playerArr.value.length) {
        playerArr.value.pop();
      }
      const { data } = await axios.get(
        "https://www.deckofcardsapi.com/api/deck/new/"
      );

      axios.get(
        "https://www.deckofcardsapi.com/api/deck/" + data.deck_id + "/shuffle/"
      );
      deckID.value = data.deck_id;
      while (playerArr.value.length) {
        playerArr.value.pop();
      }
    }

    async function startGame() {
      gameOver.value = false;
      if (deckID.value == null) {
        const { data } = await axios.get(
          "https://www.deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1"
        );
        deckID.value = data.deck_id;
      }
    }

    return {
      playerNum,
      playerArr,
      winnerID,
      getCards,
      gameOver,
      restartGame,
      startGame,
      gameOverScreen,
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
