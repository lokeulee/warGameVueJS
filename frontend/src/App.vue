<template>
  <div>
    <img alt="Vue logo" src="./assets/logo.png" />
    <h1>War Game!</h1>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import axios from "axios";

export default defineComponent({
  name: "App",
  setup() {
    // const Player = {}
    let gameOver = ref(true),
      cardOne = ref({}),
      cardTwo = ref({}),
      deckID = ref(null),
      playerOneScore = ref(0),
      playerTwoScore = ref(0);
    console.log(cardOne, cardTwo, playerOneScore, playerTwoScore);

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
    return { getDeck };
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
