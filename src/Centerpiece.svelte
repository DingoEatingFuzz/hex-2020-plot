<script>
  import {
    forceSimulation,
    forceLink,
    forceManyBody,
    forceCenter,
    forceX
  } from "d3-force";
  import Thorn from "./Thorn.svelte";

  const d3 = { forceSimulation, forceLink, forceManyBody, forceCenter, forceX };

  export let count = 7;
  export let width;
  export let height;

  const range = (min, max) => Math.random() * (max - min) + min;
  const pick = list => list[Math.floor(Math.random() * list.length)];

  let nodes = new Array(count).fill(null).map((_, id) => ({
    id,
    length: range(200, 400),
    angle: range(0, 360)
  }));

  let links = [];
  for (let currentLink = count - 1; currentLink >= 0; currentLink--) {
    const source = nodes[currentLink];
    for (let i = currentLink - 1; i >= 0; i--) {
      const target = nodes[i];
      links.push({
        source: source.id,
        target: target.id,
        weight: Math.max(source.length, target.length)
      });
    }
  }

  const simulation = d3
    .forceSimulation(nodes)
    .force(
      "link",
      d3
        .forceLink(links)
        .id(d => d.id)
        .distance(d => d.weight * 0.75)
    )
    .force("chart", d3.forceManyBody())
    .force("center", d3.forceCenter(width / 2, height / 2 + 25))
    .force("horizontalSqueeze", d3.forceX(width / 2).strength(0.2));

  simulation.tick(100);

  console.log(nodes, links);

  // Make a force-directed graph simulation
  // Extract points from it
  let points = [];
</script>

<g class="centerpiece">
  {#each nodes as p}
    <g class="thorn-container" transform="translate(0 {height}) scale(1 -1)">
      <Thorn x={p.x} y={p.y} length={p.length} angle={p.angle} />
    </g>
  {/each}
</g>
