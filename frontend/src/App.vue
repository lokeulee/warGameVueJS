<template>
  <div>
    <img alt="Vue logo" src="./assets/logo.png" />
    <h1>War Game!</h1>
    <div id="app">
      <label for="player_count">Number of players:</label>
      <select id="player_count" name="player_count" v-model="playerNum">
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
        <option value="6">6</option>
      </select>
      <div v-for="player in playerArr" :key="player.id">
        <label for="playername">{{
          "Player " + (player.id + 1) + " name: "
        }}</label>
        <input type="text" id="test" name="test" /><br /><br />
      </div>

      <input type="submit" value="Submit" />
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
      cardOne = ref({}),
      cardTwo = ref({}),
      deckID = ref(null),
      playerOneScore = ref(0),
      playerTwoScore = ref(0);
    let playerArr = ref<Player[]>([]);

    // let playerName = ref<string[]>([]);

    console.log(cardOne, cardTwo, playerOneScore, playerTwoScore);

    watchEffect(() => {
      function namePlayers() {}

      function constructArr() {
        while (playerArr.value.length) {
          playerArr.value.pop();
        }
        for (let count = 0; count <= playerNum.value - 1; count++) {
          playerArr.value[count] = { id: count, points: 0, name: "" };
        }
      }
      constructArr();
    });

    console.log(playerArr.value);

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
    getDeck();
    return { getDeck, playerNum, playerArr };
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
