<script>
  import { onMount } from "svelte";

  import Texture from "./Texture.svelte";
  import Border from "./Border.svelte";
  import Centerpiece from "./Centerpiece.svelte";

  let canvas;

  onMount(() => {
    canvas = canvas;
  });
</script>

<style>
  canvas {
    display: none;
  }
</style>

<div>
  <canvas bind:this={canvas} height={100} width={100} />
  <svg
    id="art"
    title="hex-2020-plot"
    width="900"
    height="900"
    viewBox="0 0 900 900">
    <Texture {canvas} texture="texture.png" lowDensity={30} highDensity={18} />
    <Border />
    <Centerpiece count={7} width={900} height={900} />
  </svg>
</div>
