<script>
  import Frame from "./Frame.svelte";
  import Texture from "./Texture.svelte";
  import SvgSaver from "svgsaver";

  let saver = new SvgSaver();

  function saveSVG() {
    saver.asSvg(document.querySelector("#art"));
  }
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

  .download {
    position: fixed;
    right: 0;
    top: 0;
    max-width: 200px;
  }
</style>

<main>
  <button class="download" on:click={saveSVG}>Download SVG</button>
  <div class="canvas">
    <Frame>
      <Texture texture="texture.png" lowDensity={30} highDensity={18} />
    </Frame>
  </div>
</main>
