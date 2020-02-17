<script>
  import { onMount } from "svelte";

  // Public
  export let texture;
  export let lowDensity;
  export let highDensity;

  // Private
  let points = [];
  let canvas;

  let shapes = [
    "triangleUp",
    "triangleDown",
    "parallelogramLeft",
    "parallelogramRight",
    "circle",
    "hexagon"
  ];

  class Radius {
    constructor(r) {
      this.r = r;
      this.r2 = r * r;
      this.R = 3 * this.r2;
    }
  }

  class PDD {
    constructor(width, height, texture, canvas, lr, hr) {
      // Dimensions of the distribution
      this.width = width;
      this.height = height;

      this.maxSamples = 30;

      // 2D two-tone texture denoting when a point is in
      // r1 or r2 range
      this.texture = texture;

      // The low-density radius
      this.lr = new Radius(lr);
      // The high-density radius
      this.hr = new Radius(hr);

      // Cell size MUST be based off the smaller radius
      this.cellSize = this.hr.r * Math.SQRT1_2;
      this.gridWidth = Math.ceil(width / this.cellSize);
      this.gridHeight = Math.ceil(height / this.cellSize);
      this.grid = new Array(this.gridWidth * this.gridHeight);

      this.queue = [];
      this.queueSize = 0;
      this.sampleSize = 0;

      this.canvas = canvas;
      this.ctx = this.canvas.getContext("2d");

      // Load the texture
      let img = new Image();
      this.load = new Promise((resolve, reject) => {
        img.onload = () => {
          this.ready = true;
          this.ctx.drawImage(img, 0, 0);
          this.img = this.ctx.getImageData(0, 0, 100, 100);
          // console.log(this.img);
          resolve();
        };
        img.onerror = () => {
          reject();
        };
      });
      img.src = texture;
    }

    next() {
      if (!this.ready)
        throw new Error("Cannot place points until the texture is loaded.");
      const { maxSamples, queue, width, height } = this;

      if (!this.sampleSize) {
        return this.sample(Math.random() * width, Math.random() * height);
      }

      while (this.queueSize) {
        let i = Math.floor(Math.random() * this.queueSize);
        let s = queue[i];

        // Get the color of the texture pixel proportionally under the target point
        // Just the red channel, since the texture is black and white
        let px = Math.floor((s[0] / width) * 100);
        let py = Math.floor((s[1] / height) * 100);
        let color = this.img.data[(py * 100 + px) * 4];

        // Choose the stippling density based on the color
        const sr = color === 255 ? this.lr : this.hr;

        for (let j = 0; j < maxSamples; ++j) {
          let a = 2 * Math.PI * Math.random();
          let r = Math.sqrt(Math.random() * sr.R + sr.r2);
          let x = s[0] + r * Math.cos(a);
          let y = s[1] + r * Math.sin(a);

          if (0 <= x && x < width && 0 <= y && y < height && this.far(x, y, sr))
            return this.sample(x, y);
        }

        queue[i] = queue[--this.queueSize];
        queue.length = this.queueSize;
      }
    }

    far(x, y, sr) {
      const { cellSize, gridWidth, gridHeight, grid } = this;
      const maxDist = sr.r2;

      // Translate real coordinates to grid coordinates
      let i = Math.floor(x / this.cellSize);
      let j = Math.floor(y / this.cellSize);

      // Lower and upper bounds of x and y grid cell deltas
      // given the 2r annulus constraint.
      let i0 = Math.max(i - 2, 0);
      let j0 = Math.max(j - 2, 0);
      let i1 = Math.min(i + 3, gridWidth);
      let j1 = Math.min(j + 3, gridHeight);

      // Check each square in the subgrid for a point.
      // If there is a point, and that point is less than
      // the max distance away, this (x, y) is not far.
      for (j = j0; j < j1; ++j) {
        let o = j * gridWidth;
        for (i = i0; i < i1; ++i) {
          let s = grid[o + i];
          if (s) {
            let dx = s[0] - x;
            let dy = s[1] - y;
            if (dx * dx + dy * dy < maxDist) {
              return false;
            }
          }
        }
      }

      return true;
    }

    sample(x, y) {
      var s = [x, y];
      this.queue.push(s);

      let cellX = Math.floor(x / this.cellSize);
      let cellY = Math.floor(y / this.cellSize);
      this.grid[cellY * this.gridWidth + cellX] = s;
      this.sampleSize++;
      this.queueSize++;
      return s;
    }
  }

  function geom() {
    return shapes[Math.floor(Math.random() * shapes.length)];
  }

  onMount(() => {
    console.log(texture, lowDensity, highDensity, canvas);
    if (texture && lowDensity && highDensity && canvas) {
      const pdd = new PDD(860, 860, texture, canvas, lowDensity, highDensity);

      pdd.load.then(() => {
        console.log("promise resolved");
        let p;
        while ((p = pdd.next())) {
          points.push({
            x: p[0],
            y: p[1],
            shape: geom()
          });
        }
        points = points;
      });
    }
  });
</script>

<style>
  .texture circle,
  .texture path {
    fill: transparent;
    stroke: #455;
    stroke-width: 1;
  }

  .frame {
    stroke: #fff;
    stroke-width: 1;
  }

  .texture .trace {
    fill: pink;
    stroke: transparent;
  }

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
    <g class="texture" transform="translate(25, 25)">
      {#each points as p}
        <g transform="translate({p.x - 13}, {p.y - 13}) scale(0.5, 0.5)">
          <!-- <circle class="trace" cx={13} cy={13} r="1.5" /> -->
          {#if p.shape == 'triangleUp'}
            <path d="M15 1L28.8564 25H1.14359L15 1Z" />
          {:else if p.shape == 'triangleDown'}
            <path d="M15 25L1.14359 0.999997L28.8564 1L15 25Z" />
          {:else if p.shape == 'parallelogramLeft'}
            <path d="M1 1L23 7V27L1 22V1Z" />
          {:else if p.shape == 'parallelogramRight'}
            <path d="M23 1L1 7V27L23 22V1Z" />
          {:else if p.shape == 'circle'}
            <circle cx="14" cy="14" r="13" />
          {:else}
            <path d="M12 1L23.2583 7.5V20.5L12 27L0.74167 20.5V7.5L12 1Z" />
          {/if}
        </g>
      {/each}
    </g>
    <g class="frame">
      <rect
        x="72"
        y="72"
        width="756"
        height="716"
        fill="none"
        stroke-miterlimit="10" />
      <g>
        <polygon
          points="792.44 800.6 775.75 810.23 775.75 810.23 775.75 833.36 782.02
          836.98 782.02 813.85 792.44 807.83 792.44 800.6"
          fill="none"
          stroke-miterlimit="10" />
        <polygon
          points="799.1 800.6 799.1 819.02 792.44 819.02 792.44 812.14 786.17
          815.76 786.17 839.36 792.44 842.99 792.44 824.63 799.1 824.63 799.1
          831.45 805.38 827.83 805.38 804.22 799.1 800.6"
          fill="none"
          stroke-miterlimit="10" />
        <polygon
          points="799.1 843 815.79 833.37 815.79 833.36 815.79 810.24 809.52
          806.62 809.52 829.75 799.1 835.77 799.1 843"
          fill="none"
          stroke-miterlimit="10" />
      </g>
      <g>
        <path
          d="M92.93,833.41H90.19V823.32H86.32v10.09H83.61V811.8h2.71v8.74h3.87V811.8h2.74Z"
          fill="none" />
        <path
          d="M104,833.41H95.77V811.8h8v2.81H98.48v5.55H102V823H98.48v7.58H104Z"
          fill="none" />
        <path
          d="M116.25,833.41h-3l-2.12-6.51-2.1,6.51h-3l3.65-11.06-3.49-10.55h3l1.88,6,2-6h3l-3.48,10.55Z"
          fill="none" />
        <path
          d="M124.12,808.66a3.91,3.91,0,0,1-.35,1.6,22.25,22.25,0,0,1-1.22,2.17,19.6,19.6,0,0,0-1.7,3.1,9.62,9.62,0,0,0-.79,2.57h4.06V820h-6.19a12.58,12.58,0,0,1,.84-4.46,15,15,0,0,1,1.89-3.64,15.57,15.57,0,0,0,1.2-1.94,3,3,0,0,0,.39-1.27,1.94,1.94,0,0,0-.35-1.16,1.15,1.15,0,0,0-.85-.36,1.23,1.23,0,0,0-.84.29,1.44,1.44,0,0,0-.43,1h-1.85a3,3,0,0,1,.89-2.16,3,3,0,0,1,2.23-.94,2.87,2.87,0,0,1,2.21.93A3.38,3.38,0,0,1,124.12,808.66Z"
          fill="none" />
        <path
          d="M129.22,805.33a2.7,2.7,0,0,1,2.66,1.76,21,21,0,0,1,0,11.33,2.86,2.86,0,0,1-5.33,0,21.22,21.22,0,0,1,0-11.33A2.72,2.72,0,0,1,129.22,805.33Zm0,2a1.2,1.2,0,0,0-1.06.77c-.27.5-.44,2-.49,4.61a14.8,14.8,0,0,0,.49,4.61,1.19,1.19,0,0,0,1.06.76,1.21,1.21,0,0,0,1.07-.76,14.8,14.8,0,0,0,.49-4.61c-.05-2.57-.22-4.11-.49-4.61A1.22,1.22,0,0,0,129.18,807.35Z"
          fill="none" />
        <path
          d="M140.46,808.66a3.76,3.76,0,0,1-.35,1.6,22.25,22.25,0,0,1-1.22,2.17,18.77,18.77,0,0,0-1.69,3.1,9.71,9.71,0,0,0-.8,2.57h4.06V820h-6.19a12.58,12.58,0,0,1,.84-4.46,15,15,0,0,1,1.89-3.64,15.84,15.84,0,0,0,1.21-1.94,3.09,3.09,0,0,0,.38-1.27,1.88,1.88,0,0,0-.35-1.16,1.14,1.14,0,0,0-.85-.36,1.23,1.23,0,0,0-.84.29,1.48,1.48,0,0,0-.43,1h-1.85a3,3,0,0,1,.89-2.16,3,3,0,0,1,2.23-.94,2.87,2.87,0,0,1,2.21.93A3.33,3.33,0,0,1,140.46,808.66Z"
          fill="none" />
        <path
          d="M145.56,805.33a2.72,2.72,0,0,1,2.67,1.76,21.22,21.22,0,0,1,0,11.33,2.86,2.86,0,0,1-5.34,0,21.22,21.22,0,0,1,0-11.33A2.72,2.72,0,0,1,145.56,805.33Zm0,2a1.23,1.23,0,0,0-1.06.77c-.27.5-.43,2-.49,4.61a15.14,15.14,0,0,0,.49,4.61,1.13,1.13,0,0,0,2.13,0,21.5,21.5,0,0,0,0-9.22A1.22,1.22,0,0,0,145.52,807.35Z"
          fill="none" />
      </g>
    </g>
  </svg>
</div>
