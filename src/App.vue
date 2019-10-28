<script>
import GameLogic from './components/GameLogic';
import vueLogo from './assets/cat.jpg';

const TOP_MARGIN = 40;

export default {
  name: "app",
  components: {
    GameLogic
  },
  data() {
    return {
      imageSource: vueLogo,
      image: null,
      topMargin: TOP_MARGIN,
      topMarginStyle: {
        '--top-margin': `${TOP_MARGIN}px`
      }
    }
  },
  mounted() {
    this.image = document.querySelector('img');
  }
};
</script>

<template>
  <div id="app" :style="topMarginStyle">
    <div class="main-image">
      <img class="ui image" :src="imageSource" alt="Image to filter">
    </div>
    <GameLogic :image="image" :topMargin="topMargin"></GameLogic>
  </div>
</template>

<style>

#app img {
  position: absolute;
  top: var(--top-margin);
  left: 0;
}

</style>
