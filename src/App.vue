<template>
  <div class="d-flex justify-content-center align-items-center mt-5">
    <Board />
  </div>
</template>

<script>
import Board from './components/Board.vue'

export default {
  name: 'App',
  components: {
    Board
  },
};
</script>