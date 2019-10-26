<template>
	<div>
    <v-app-bar
      class="header-game" 
      color="grey darken-4">
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn
            v-on="on"
            v-on:click="$emit('reset-position')"
            text
            icon
            color="white"
          >
            <v-icon color="white">mdi-crosshairs-gps</v-icon>
          </v-btn>
        </template>
        <span>Reset position</span>
      </v-tooltip>
      <div 
        id="countdown-timer"
        v-if="remainingTime != null && remainingTime != 0"
      >
        <span id="countdown-text">{{ getCountdownText }}</span>
      </div>
      <div class="flex-grow-1"></div>
      <div class="round-score-container">
        <span class="sub-text">ROUND: </span>
      </div>
      <div>
        <span class="main-text">{{ round }} / 5</span>
      </div>
      <div class="round-score-container">
        <span class="sub-text">SCORE: </span>
      </div>
      <div>
        <span class="main-text">{{ score }} km away</span>
      </div>
    </v-app-bar>
  </div>
</template>

<script>
export default {
    props: [
        'score',
        'round',
        'remainingTime'
    ],
    computed: {
        getCountdownText() {
            var minutes = Math.floor(this.remainingTime / 60);
            var seconds = this.remainingTime % 60;
            if (minutes < 10) { minutes = '0' + minutes; }
            if (seconds < 10) { seconds = '0' + seconds; }
            return minutes + ':' + seconds;
        },
    },
};
</script>

<style scoped>
  .header-game {
    z-index: 3;
    opacity: 0.8;
  }

  .toolbar-title {
    color: white;
  }

  .round-score-container {
    padding: 0 10px 0 40px;
  }

  .main-text, #countdown-text {
    color: white;
  }

  .sub-text {
    color: #616161;
  }

  @media (max-width: 450px) {
    .main-text, .sub-text, #countdown-text {
      font-size: 14px;
    }
  }
</style>
