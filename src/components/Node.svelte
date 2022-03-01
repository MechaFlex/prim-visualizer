<script lang="ts">
  export let x: number
  export let y: number
</script>

<div class="dotwrapper" style="left: {x}px; top: {y}px">
  <div class="dot"></div>
</div>

<style>
  .dotwrapper {
    position: absolute;
    display: grid;
    place-items: center;
    width: 2rem;
    height: 2rem;
  }

  .dot {
    width: 30%;
    height: 30%;
    border-radius: 2rem;
    background-color: black;
    transition: all 500ms;
  }

  .dotwrapper:hover .dot {
    width: 60%;
    height: 60%;
    transition: all 100ms;
  }
</style>