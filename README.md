# HEX 2020 Plot

In 2020, the HashiCorp Employee Exchange will be in Atlanta.

I've been thinking a lot about travel lately, and how there are wrong ways and right ways to do it. Every year countless people representing countless organizations go to countless cities for events. This is just another one of them. We'll spend most of our time in interior rooms of hotels where our exact geospatial coordinates are unimportant. Much like airports, these rooms are sort of untethered from their surroundings.

And yet, each of these cities are real places. With residents, and history, and culture, and authenticity. Maybe this is all because Portland's locally loved 9th and Alder food cart pod was evicted when the plot underneath them was sold to eventually become a Ritz Carlton of all things, but I've been keen to travel a little more mindfully even for work.

With that in mind, I drew inspiration from local artists and Native traditions.

### Background Texture

The background texture is plotted with a Sky colored Stardust Sakura Gelly Roll pen. This creates a subtle color on black with a shimmer.

The design is inspired by Native basketry indigenous to the Southeast region, and the individual shapes pay homage to Native beadwork.

This pattern is implemented as a dual density Poisson disc distribution. The density for a point is determined by reading the proportional pixel value from static texture. White means low density and black means high density.

### Centerpiece

The centerpiece of seven thorns is plotted with a white Sakura Gelly Roll pen. The design is inspired by local sculptor [Ayokunle Odeleye](http://www.odeleyesculpturestudios.com/index.html). He in turn takes inspiration from ancient Malian art.

Each thorn is implemented using bezier curves with variable length, width, center spline positioning, and bend. To give each thorn depth, each side is textured differently. One with regular strokes and the other with erratic strokes (which are just regular strokes with a slight amount of jitter in the start and end points along curves).

The arrangement of the thorns is determined by creating a force directed graph with node connected and weighted by the size of the thorn.

### Incorporating the HashiCorp Brand

The white-on-black design as well as the framing and labeling is a direct nod to the HEX plot for HEX^3. The number seven is also meaningful to the company.

## Development

```bash
npm install
npm run dev
```

The site is hosted at [localhost:5000](http://localhost:5000).

## Production

This is kinda nice since pages reload faster

```bash
npm run build
npm run start
```

The site will again be hosted at [localhost:5000](http://localhost:5000).
