<script>
    import { onMount } from 'svelte';
    import Webpage from './Webpage.svelte';
  
    let webpages = [];
    const numPages = 10;
    let centerX, centerY;
    const iconRadius = 40;
  
    const positions = [
      { x: 249.5000192540643, y: 33.89903043165276 },
      { x: 409.6509864586941, y: 2.365906563291471 },
      { x: 590.26141316703, y: 0 },
      { x: 293.21812988699224, y: 182.6686421804885 },
      { x: 406.7913581924273, y: 120.6841541710913 },
      { x: 581.4871787907978, y: 136.2767292134302 },
      { x: 320.0751161990354, y: 310.78970922838016 },
      { x: 443.03237339963, y: 313.8660780792031 },
      { x: 553.702150068209, y: 276.67774720690394 },
      { x: 298.70906577268704, y: 450.4822200693375 }
    ];
  
    const arrows = [
      { from: 0, to: 1 },
      { from: 1, to: 2 },
          { from: 1, to: 4 },
      { from: 2, to: 4 },
      { from: 3, to: 4 },
      { from: 4, to: 5 },
      { from: 5, to: 8 },
      { from: 6, to: 9 },
      { from: 7, to: 4 },
          { from: 7, to: 6 },
      { from: 8, to: 7 },
    ];
  
    onMount(() => {
      centerX = window.innerWidth / 2;
      centerY = window.innerHeight / 2;
  
      webpages = Array.from({ length: numPages }, (_, i) => ({
        id: i,
        x: centerX,
        y: centerY,
      }));
    });
  
    function scatter() {
      webpages = webpages.map((page, i) => ({
        ...page,
        x: positions[i].x,
        y: positions[i].y
      }));
    }
  
    function calculateArrowPosition(from, to, progress) {
      const fromCenterX = from.x + 30;
      const fromCenterY = from.y + 30;
      const toCenterX = to.x + 30;
      const toCenterY = to.y + 30;
  
      const dx = toCenterX - fromCenterX;
      const dy = toCenterY - fromCenterY;
      const distance = Math.sqrt(dx * dx + dy * dy);
      const offsetX = dx * (iconRadius / distance);
      const offsetY = dy * (iconRadius / distance);
  
      const currentX = fromCenterX + offsetX + progress * (toCenterX - fromCenterX - 2.5 * offsetX);
      const currentY = fromCenterY + offsetY + progress * (toCenterY - fromCenterY - 2.5 * offsetY);
  
      return {
        x1: fromCenterX + offsetX,
        y1: fromCenterY + offsetY,
        x2: currentX,
        y2: currentY
      };
    }
  
    let arrowsVisible = false;
  
    function animateArrows() {
      arrowsVisible = false;
      setTimeout(() => {
        arrowsVisible = true;
        arrows.forEach(({ from, to }) => {
          animateArrow(from, to);
        });
      }, 1000); // Delay to wait for the webpages to scatter
    }
  
    function animateArrow(fromIndex, toIndex) {
      const from = webpages[fromIndex];
      const to = webpages[toIndex];
      const duration = 1000; // duration of the animation in milliseconds
      const startTime = performance.now();
  
      function step(currentTime) {
        const progress = Math.min((currentTime - startTime) / duration, 1);
        const { x1, y1, x2, y2 } = calculateArrowPosition(from, to, progress);
  
        const arrowLine = document.querySelector(`#arrow-${fromIndex}-${toIndex}`);
        arrowLine.setAttribute('x1', x1);
        arrowLine.setAttribute('y1', y1);
        arrowLine.setAttribute('x2', x2);
        arrowLine.setAttribute('y2', y2);
  
        if (progress < 1) {
          requestAnimationFrame(step);
        }
      }
  
      requestAnimationFrame(step);
    }
  </script>
  
  <style>
    .container {
      position: relative;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
    }
  
    svg {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
  
    .arrow {
      stroke: black;
      stroke-width: 2;
      marker-end: url(#arrowhead);
    }
  
    @keyframes draw {
      from {
        stroke-dasharray: 0, var(--line-length);
      }
      to {
        stroke-dasharray: var(--line-length), 0;
      }
    }
  </style>
  
  <div class="container">
    {#each webpages as page (page.id)}
      <Webpage x={page.x} y={page.y} />
    {/each}
    <svg>
      <defs>
        <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="0" refY="3.5" orient="auto">
          <polygon points="0 0, 10 3.5, 0 7" />
        </marker>
      </defs>
      {#if arrowsVisible}
        {#each arrows as { from, to }}
          {#if webpages[from] && webpages[to]}
            <line
              id={"arrow-" + from + "-" + to}
              class="arrow"
            />
          {/if}
        {/each}
      {/if}
    </svg>
  </div>
  
  <button on:click={() => { scatter(); animateArrows(); }}>
    Scatter
  </button>
  