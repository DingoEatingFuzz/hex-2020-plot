<script>
  import Frame from "./Frame.svelte";
  import Texture from "./Texture.svelte";
</script>

<style>
  main {
    width: 100%;
    height: 100vh;
    background: #55555f;
  }

  .canvas {
    padding-top: 30px;
    display: flex;
    justify-content: center;
  }
</style>

<main>
  <div class="canvas">
    <Frame>
      <Texture texture="texture.png" lowDensity={30} highDensity={18} />
    </Frame>
  </div>
</main>
