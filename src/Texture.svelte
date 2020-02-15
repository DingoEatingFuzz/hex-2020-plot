<script>
  class Radius {
    constructor(r) {
      this.r = r;
      this.r2 = r * r;
      this.R = 3 * this.r2;
    }
  }

  class PDD {
    constructor(width, height, texture, lr, hr) {
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
    }

    next() {
      const { maxSamples, queue, width, height } = this;
      const sr = this.hr; // TODO: this should be chosen based on the texture

      if (!this.sampleSize) {
        return this.sample(Math.random() * width, Math.random() * height);
      }

      while (this.queueSize) {
        let i = Math.floor(Math.random() * this.queueSize);
        let s = queue[i];

        for (let j = 0; j < maxSamples; ++j) {
          let a = 2 * Math.PI * Math.random();
          let r = Math.sqrt(Math.random() * sr.R + sr.r2);
          let x = s[0] + r * Math.cos(a);
          let y = s[1] + r * Math.sin(a);

          if (0 <= x && x < width && 0 <= y && y < height && this.far(x, y))
            return this.sample(x, y);
        }

        queue[i] = queue[--this.queueSize];
        queue.length = this.queueSize;
      }
    }

    far(x, y) {
      const { cellSize, gridWidth, gridHeight, grid } = this;
      const maxDist = this.hr.r2;

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

  const pdd = new PDD(900, 900, null, 40, 20);

  let points = [];

  let p;
  while ((p = pdd.next())) {
    points.push(p);
  }
</script>

<style>
  svg circle {
    fill: #fff;
  }
</style>

<svg width="900" height="900" viewBox="0 0 900 900">
  {#each points as [x, y]}
    <circle cx={x} cy={y} r="1.5" />
  {/each}
</svg>
