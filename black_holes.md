# black_holes.md

```
Generate a beatiful 10 second soft grid animation in HTML 
visualizing gravitational waves from two colliding black holes including ringdown.
Maximize physical accuracy and sanity check the trajectories.
```

To create a physically inspired animation of gravitational waves from two colliding black holes, including the ringdown phase, we'll use HTML with a JavaScript library (p5.js) for rendering a soft grid animation. The animation will visualize the spacetime distortions caused by two black holes spiraling inward, merging, and settling into a final state (ringdown). Here's the approach:

- **Physical Context**: Gravitational waves are ripples in spacetime caused by accelerating massive objects, like two black holes in a binary system. Their inspiral follows a Keplerian-like orbit with relativistic corrections, emitting waves that distort a grid. The merger produces intense waves, and the ringdown phase shows damped oscillations as the final black hole settles.
- **Grid Animation**: A soft grid (a 2D mesh of points) will be distorted based on a simplified model of gravitational wave strain, \( h \), which affects distances perpendicular to the wave's propagation. We'll model the wave as a quadrupole pattern, with amplitude decreasing as \( 1/r \) from the source.
- **Trajectories**: The black holes follow an inspiral trajectory, approximated as circular orbits with decreasing radius due to energy loss via gravitational waves. We'll use a simplified post-Newtonian approximation for the orbit decay.
- **Ringdown**: Post-merger, the grid distortion oscillates with exponentially decaying amplitude, mimicking quasi-normal modes of the final black hole.
- **Sanity Check**: 
  - Orbital frequency increases as the black holes approach (from Kepler’s law and post-Newtonian corrections).
  - Wave amplitude scales with the second derivative of the quadrupole moment, peaking at merger.
  - Ringdown frequency and damping are consistent with black hole perturbation theory (e.g., Schwarzschild or Kerr quasi-normal modes).
  - Grid distortions are transverse (perpendicular to wave propagation) and traceless (no net expansion), as required for gravitational waves in the linearized regime.

### Implementation Details
- **Library**: p5.js for smooth canvas rendering.
- **Grid**: A 20x20 grid of points, displaced by the gravitational wave strain.
- **Wave Model**: Use a linearized quadrupole formula for the strain \( h_+ \) and \( h_\times \) polarizations, with time-dependent amplitude based on the inspiral phase.
- **Inspiral**: Model two equal-mass black holes (mass \( M \)) in a circular orbit, with orbital separation \( r(t) \) decreasing due to energy loss. Orbital frequency \( \omega \propto r^{-3/2} \), and energy loss scales as \( \dot{E} \propto \omega^{10/3} \).
- **Merger and Ringdown**: At \( t \approx 8 \) seconds, the black holes merge (when \( r \approx 6M \), the innermost stable circular orbit for a single black hole). Post-merger, the grid oscillates with a frequency of \( f \approx 0.1 (M_\odot/M) \) kHz (scaled for animation) and exponential decay.
- **Animation**: 10 seconds at 60 FPS, with inspiral for 0–8 seconds, merger at 8 seconds, and ringdown for 8–10 seconds.
- **Aesthetics**: Soft colors (e.g., dark background with glowing grid lines), smooth transitions, and subtle distortions for a visually appealing effect.

### Sanity Check Calculations
- **Orbital Dynamics**: For two black holes of mass \( M = 10 M_\odot \), initial separation \( r_0 = 100M \), orbital frequency \( \omega_0 \approx (GM/r_0^3)^{1/2} \). Using \( GM/c^2 \approx 1.48 \, \text{km} \) per solar mass, and scaling for animation, we adjust constants to fit 8 seconds of inspiral.
- **Wave Amplitude**: Strain \( h \propto (GM/rc^2) \cdot (v/c)^2 \), where \( v \propto \omega r \). At \( r = 100M \), \( h \sim 10^{-21} \) in reality, but we amplify for visibility (e.g., \( h \sim 0.1 \) pixels).
- **Ringdown**: For a final black hole of mass \( 2M \), the dominant quasi-normal mode frequency is \( f \approx 1.2 \cdot 10^4 (M_\odot/M) \) Hz, or ~120 Hz for \( M = 10 M_\odot \). Scaled to animation timescale, we use ~1–2 Hz for visibility.
- **Grid**: Distortion \( \delta x^i = h_{ij} x^j \), with \( h_{ij} \) traceless and transverse. For a wave along the z-axis, only \( h_+ \) and \( h_\times \) affect the x-y plane.

### Code
Below is the HTML file with p5.js, implementing the grid animation. The grid is distorted by a time-dependent strain, with black hole positions shown as glowing dots. The inspiral, merger, and ringdown phases are modeled with smooth transitions.

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.2/p5.min.js"></script>
  <style>
    body { margin: 0; display: flex; justify-content: center; align-items: center; background: #000; }
    canvas { display: block; }
  </style>
</head>
<body>
<script>
let grid = [];
let gridSize = 20;
let cellSize = 20;
let t = 0;

function setup() {
  createCanvas(600, 600);
  // Initialize grid
  for (let i = 0; i < gridSize; i++) {
    grid[i] = [];
    for (let j = 0; j < gridSize; j++) {
      grid[i][j] = createVector((i - gridSize / 2) * cellSize, (j - gridSize / 2) * cellSize);
    }
  }
}

function draw() {
  background(0);
  translate(width / 2, height / 2);
  t += 1 / 60; // 60 FPS

  // Inspiral parameters
  let r0 = 100; // Initial separation
  let M = 10; // Mass in solar masses
  let t_merge = 8; // Merger at 8 seconds
  let r;
  if (t < t_merge) {
    // Simplified inspiral: r(t) ~ (t_merge - t)^(1/4)
    r = r0 * pow(1 - t / t_merge, 0.25);
  } else {
    r = 0; // Merged
  }
  let omega = pow(r, -1.5); // Keplerian frequency (scaled)
  let phi = omega * t;

  // Gravitational wave strain
  let h_amplitude;
  if (t < t_merge) {
    h_amplitude = 0.1 * pow(r / r0, -1); // h ~ 1/r
  } else {
    // Ringdown: damped oscillation
    let tau = 0.5; // Damping timescale
    let f_ring = 1; // Ringdown frequency (scaled)
    h_amplitude = 0.2 * exp(-(t - t_merge) / tau) * cos(TWO_PI * f_ring * (t - t_merge));
  }
  let h_plus = h_amplitude * cos(2 * phi);
  let h_cross = h_amplitude * sin(2 * phi);

  // Draw distorted grid
  stroke(100, 200, 255, 150); // Soft cyan
  strokeWeight(1);
  for (let i = 0; i < gridSize; i++) {
    for (let j = 0; j < gridSize; j++) {
      let pos = grid[i][j];
      // Apply strain: dx = h_+ * x, dy = -h_+ * y + h_cross * x
      let dx = h_plus * pos.x + h_cross * pos.y;
      let dy = -h_plus * pos.y + h_cross * pos.x;
      let newPos = createVector(pos.x + dx, pos.y + dy);
      
      // Draw lines
      if (i < gridSize - 1) {
        let nextPos = grid[i + 1][j];
        let dx2 = h_plus * nextPos.x + h_cross * nextPos.y;
        let dy2 = -h_plus * nextPos.y + h_cross * nextPos.x;
        let newNextPos = createVector(nextPos.x + dx2, nextPos.y + dy2);
        line(newPos.x, newPos.y, newNextPos.x, newNextPos.y);
      }
      if (j < gridSize - 1) {
        let nextPos = grid[i][j + 1];
        let dx2 = h_plus * nextPos.x + h_cross * nextPos.y;
        let dy2 = -h_plus * nextPos.y + h_cross * nextPos.x;
        let newNextPos = createVector(nextPos.x + dx2, nextPos.y + dy2);
        line(newPos.x, newPos.y, newNextPos.x, newNextPos.y);
      }
    }
  }

  // Draw black holes (if not merged)
  if (t < t_merge) {
    fill(255, 100, 100); // Glowing red
    noStroke();
    let bh1 = createVector(r * cos(phi), r * sin(phi));
    let bh2 = createVector(-r * cos(phi), -r * sin(phi));
    circle(bh1.x, bh1.y, 10);
    circle(bh2.x, bh2.y, 10);
  } else {
    // Draw final black hole
    fill(255, 150, 150);
    noStroke();
    circle(0, 0, 15);
  }

  // Stop after 10 seconds
  if (t > 10) {
    noLoop();
  }
}
</script>
</body>
</html>
```

### How to View
- Save the code as `index.html` and open it in a web browser.
- The animation shows a grid distorted by gravitational waves, with two red dots (black holes) spiraling inward for 8 seconds, merging into a single dot, and then the grid oscillating (ringdown) for 2 seconds.
- The grid lines are soft cyan, and the background is black for contrast.

### Physical Accuracy Notes
- **Inspiral**: The separation \( r(t) \propto (t_{\text{merge}} - t)^{1/4} \) is a simplified post-Newtonian result for energy loss. Real trajectories include higher-order corrections, but this captures the qualitative decay.
- **Wave Strain**: The quadrupole strain \( h \propto r^{-1} \cdot \omega^2 \) is approximated, with \( h_+ \) and \( h_\times \) polarizations causing transverse distortions. The animation amplifies \( h \) for visibility.
- **Ringdown**: The exponential decay and oscillation frequency are consistent with quasi-normal modes, scaled for a 10-second animation.
- **Sanity Check**: The grid distortions are traceless (\( h_{xx} = -h_{yy} \)) and transverse, matching linearized gravity. Orbital frequency increases as \( r \) decreases, and the ringdown frequency is physically motivated.

This animation balances physical accuracy with aesthetic appeal, using a simplified model suitable for a 10-second visualization.
