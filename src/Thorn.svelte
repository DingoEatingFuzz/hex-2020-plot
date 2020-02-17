<script>
  import { onMount } from "svelte";

  export let x = 0;
  export let y = 0;
  export let angle = 0;

  // Parameters
  let length = 300;
  let width = 50;
  let center = 30;
  let centerOffset = 20;
  let offset = 0;
  let bend = -20;

  // Bindings
  let innerCurve;
  let outerCurve;
  let middleCurve;

  let orderedStrokes = [];
  let erraticStrokes = [];

  onMount(() => {
    // Total length of each curve
    const innerLength = innerCurve.getTotalLength();
    const outerLength = outerCurve.getTotalLength();
    const middleLength = middleCurve.getTotalLength();

    const spacing = 8;

    // Ordered strokes
    // Evenly space lines from the outer curve to the middle curve to
    // create an ordered hatching pattern.
    for (let d = spacing; d < innerLength - spacing / 2; d += spacing) {
      const p1 = outerCurve.getPointAtLength(d);
      const p2 = middleCurve.getPointAtLength((d / outerLength) * middleLength);
      orderedStrokes.push({
        x1: p1.x,
        x2: p2.x,
        y1: p1.y,
        y2: p2.y
      });
    }

    // Erratic strokes
    // Similar to the ordered strokes, except there are twice as many lines
    // and the delta along the curves are jittered to create irregularity and overlap.
    for (let d = spacing / 2; d < outerLength - spacing / 2; d += spacing / 2) {
      const dr = d + Math.random() * (spacing * 2) - spacing;
      const dr2 = d + Math.random() * (spacing * 2) - spacing;
      const p1 = innerCurve.getPointAtLength(dr);
      const p2 = middleCurve.getPointAtLength(
        (dr2 / innerLength) * middleLength
      );
      erraticStrokes.push({
        x1: p1.x,
        x2: p2.x,
        y1: p1.y,
        y2: p2.y
      });
    }

    // Force svelte to re-render
    orderedStrokes = orderedStrokes;
    erraticStrokes = erraticStrokes;
  });
</script>

<style>
  path,
  line {
    stroke: #fff;
    stroke-width: 1;
    fill: transparent;
  }
</style>

<g class="thorn" transform="translate({x}, {y}) rotate({angle})">
  <path
    bind:this={innerCurve}
    d="M 0 0 Q {length / 2}
    {-width / 4 - offset + bend - width / 2}
    {length}
    {-width / 2 - offset}" />
  <path
    bind:this={outerCurve}
    d="M 0 0 Q {length / 2}
    {width / 4 - offset + bend}
    {length}
    {width / 2 - offset}" />
  <path
    bind:this={middleCurve}
    d="M 0 0 Q {(length + centerOffset) / 2}
    {(width / 2 - center) / 2 - offset + bend - width / 4}
    {length + centerOffset}
    {width / 2 - center - offset}" />
  <path
    d="M {length}
    {-width / 2 - offset} L {length + centerOffset}
    {width / 2 - center - offset} L {length}
    {width / 2 - offset}" />
  <g class="ordered-strokes">
    {#each orderedStrokes as s}
      <line x1={s.x1} y1={s.y1} x2={s.x2} y2={s.y2} />
    {/each}
  </g>
  <g class="erratic-strokes">
    {#each erraticStrokes as s}
      <line x1={s.x1} y1={s.y1} x2={s.x2} y2={s.y2} />
    {/each}
  </g>
</g>
