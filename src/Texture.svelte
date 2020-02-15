<script>
  import { onMount } from "svelte";

  // Public
  export let texture;
  export let lowDensity;
  export let highDensity;

  // Private
  let points = [];
  let canvas;

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

  onMount(() => {
    console.log(texture, lowDensity, highDensity, canvas);
    if (texture && lowDensity && highDensity && canvas) {
      const pdd = new PDD(900, 900, texture, canvas, lowDensity, highDensity);

      pdd.load.then(() => {
        console.log("promise resolved");
        let p;
        while ((p = pdd.next())) {
          points.push(p);
        }
        points = points;
      });
    }
  });
</script>

<style>
  svg circle {
    fill: #fff;
  }
  canvas {
    display: none;
  }
</style>

<div>
  <canvas bind:this={canvas} height={100} width={100} />
  <svg width="900" height="900" viewBox="0 0 900 900">
    {#each points as [x, y]}
      <circle cx={x} cy={y} r="1.5" />
    {/each}
  </svg>
</div>
