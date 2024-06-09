<script>
  import Scrolly from "./Scrolly.svelte";
  import { select } from "d3-selection";
  import { fly } from 'svelte/transition';
  import Webpage from './Webpage.svelte';
  import { onMount } from 'svelte';
  import App from './App.svelte';

  const webpage = "assets/images/webpage.svg";
  let xPos = 0;

  function moveObject() {
    xPos += 100;
  }

  let value;
  let webpages = [];
  const numPages = 8;
  let centerX, centerY;
  const iconRadius = 40;
  let showBacklinks = false;
  let showProbabilities = false;
  const walkSteps = 100;
  let probabilities = Array(numPages).fill(0);

  const positions = [
    { x: 249.5000192540643, y: 33.89903043165276 },
    { x: 409.6509864586941, y: 2.365906563291471 },
    { x: 590.26141316703, y: 0 },
    { x: 293.21812988699224, y: 182.6686421804885 },
    { x: 406.7913581924273, y: 120.6841541710913 },
    { x: 581.4871787907978, y: 136.2767292134302 },
    { x: 320.0751161990354, y: 310.78970922838016 },
    { x: 443.03237339963, y: 313.8660780792031 },
  ];

  const arrows = [
    { from: 0, to: 1 },
    { from: 1, to: 2 },
    { from: 1, to: 4 },
    { from: 2, to: 4 },
    { from: 3, to: 0 },
    { from: 4, to: 3 },
    { from: 4, to: 5 },
    { from: 4, to: 7 },
    { from: 5, to: 2 },
    { from: 6, to: 3 },
    { from: 7, to: 6 },
  ];

  const step5Arrows = [
    { from: 0, to: 1 },
    { from: 1, to: 4 },
    { from: 6, to: 2 },
    { from: 3, to: 0 },
    { from: 4, to: 3 },
    { from: 2, to: 5 },
    { from: 7, to: 6 },
    { from: 5, to: 7 },
  ];

  onMount(() => {
    centerX = window.innerWidth / 2;
    centerY = window.innerHeight / 2;

    webpages = Array.from({ length: numPages }, (_, i) => ({
      id: i,
      x: centerX,
      y: centerY,
      backlinks: 0
    }));
  });

  function scatter() {
    webpages = webpages.map((page, i) => ({
      ...page,
      x: positions[i].x,
      y: positions[i].y
    }));
    console.log("Scattered webpages positions:", webpages);
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

  function animateArrows(arrowsArray) {
    arrowsVisible = false;
    setTimeout(() => {
      arrowsVisible = true;
      arrowsArray.forEach(({ from, to }) => {
        animateArrow(from, to, 1000);
        webpages[to].backlinks += 1;
      });

      setTimeout(() => {
        showBacklinks = true;
      }, 1200); // Adjust this delay as needed
    }, 1000); // Delay to wait for the webpages to scatter
  }

  function animateArrow(fromIndex, toIndex, duration, color = 'black') {
    return new Promise((resolve) => {
      const from = webpages[fromIndex];
      const to = webpages[toIndex];
      const startTime = performance.now();

      function step(currentTime) {
        const progress = Math.min((currentTime - startTime) / duration, 1);
        const { x1, y1, x2, y2 } = calculateArrowPosition(from, to, progress);

        const arrowLine = document.querySelector(`#arrow-${fromIndex}-${toIndex}`);
        if (arrowLine) {
          arrowLine.setAttribute('x1', x1);
          arrowLine.setAttribute('y1', y1);
          arrowLine.setAttribute('x2', x2);
          arrowLine.setAttribute('y2', y2);
          arrowLine.setAttribute('stroke', color);
        }

        if (progress < 1) {
          requestAnimationFrame(step);
        } else {
          resolve();
        }
      }

      requestAnimationFrame(step);
    });
  }

  async function randomWalk(arrowsp, dampingFactor = 1) {
  let currentPage = Math.floor(Math.random() * numPages);
  const visitCounts = Array(numPages).fill(0);

  for (let step = 0; step < walkSteps; step++) {
    visitCounts[currentPage]++;
    if (Math.random() < dampingFactor) {
      const outgoingArrows = arrowsp.filter(arrow => arrow.from === currentPage);
      if (outgoingArrows.length > 0) {
        const { to } = outgoingArrows[Math.floor(Math.random() * outgoingArrows.length)];
        await animateArrow(currentPage, to, 100, 'red');
        currentPage = to;
      } else {
        // If there are no outgoing arrows, perform a random jump
        currentPage = Math.floor(Math.random() * numPages);
      }
    } else {
      // Perform a random jump to any page
      currentPage = Math.floor(Math.random() * numPages);
    }
  }

  probabilities = visitCounts.map(count => count / walkSteps);

  setTimeout(() => {
    showProbabilities = true;
  }, walkSteps * 10); // Adjust the delay to match the duration of the walk animation
}


  function resetState() {
    arrowsVisible = false;
    showBacklinks = false;
    showProbabilities = false;
    webpages = webpages.map(page => ({
      ...page,
      backlinks: 0
    }));
  }

  $: if (typeof value !== "undefined") target2event[value]();

  const target2event = {
    0: () => {
      scatter();
    },
    1: () => {
      animateArrows(arrows);
    },
    2: () => {
      randomWalk(arrows);
    },
    3: () => {
      // Do nothing
    },
    4: () => {
      // resetState();
      // scatter();        
      // animateArrows(step5Arrows);
      // randomWalk(step5Arrows, 0.85);
      // resetState();
      // scatter();        
      // animateArrows(step5Arrows);
      console.log(4);
    },
    5: () => {
      resetState();
      scatter();        
      animateArrows(step5Arrows);
      randomWalk(step5Arrows, 0.85);
      resetState();
      scatter();        
      animateArrows(step5Arrows);
      console.log(5);
    }
  };

  $: steps = [
    `<h1 class='step-title'>Step 1</h1>
       <br><br>
      <p>
       We can think of the web as a network of pages. A bot crawls the internet, traversing links to gather webpages. But now what? We need to be able to organize this mass of data. We want to return the most useful pages to the user based on their search query. Our first task is to find the most important webpages.
      </p>`,
    `<h1 class='step-title'>Step 2</h1>
      <p>
        The simplest way to rank all of these pages would be to sort them by the number of pages that link to them, or backlinks.
        </p>`,
    `<h1 class='step-title'>Step 3</h1>
      <p>
        We can improve this by considering the likelihood of a user clicking on a link. We can mimic real user behavior by taking a random walk on the graph of the web, the higher the amount of backlinks a page has, the more likely we are to land on it.
        </p>`,
    `<h1 class='step-title'>Step 4</h1>
      <p>
        With this we have found a way to differentiate important pages from less important ones. This is useful in filtering out spam and irrelevant pages.         However, there's one rather large issue with what we have so far. It is possible to get stuck as we traverse the web with links. We could get in to a dead end, or be going around in circles infinitely. We need to account for this.
        </p>`,
    `<h1 class='step-title'>Step 5</h1>
      <p>
        PageRank introduces a "damping factor" to account for this. For a damping factor of 0.85, we have an 85% chance of following a link, and a 15% chance of jumping to a random page. This is key, as it ensures that we can always escape from a dead end/loop.
        </p>`,
  ];
</script>

<section>
  <!-- scroll container -->
  <div class="section-container">
    <div class="steps-container">
      <Scrolly bind:value>
        {#each steps as text, i}
          <div class="step" class:active={value === i}>
            <div class="step-content">{@html text}</div>
          </div>
        {/each}
        <!-- <div class="spacer" /> -->
        <div class="step" class:active={value === steps.length}>
          <!-- <div class="step-content">End of the journey.</div> -->
        </div>
      </Scrolly>
    </div>
    <div class="charts-container">
      <div class="chart-one">
        <svg>
          <defs>
            <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="0" refY="3.5" orient="auto">
              <polygon points="0 0, 10 3.5, 0 7" />
            </marker>
          </defs>
          {#if arrowsVisible}
            {#each value === 4 ? step5Arrows : arrows as { from, to }}
              {#if webpages[from] && webpages[to]}
                <line
                  id={"arrow-" + from + "-" + to}
                  class="arrow"
                />
              {/if}
            {/each}
          {/if}
          {#if showBacklinks}
            {#each webpages as page (page.id)}
              <text x={page.x} y={page.y + 90} fill="black">{page.backlinks}</text>
            {/each}
          {/if}
          {#if showProbabilities}
            {#each webpages as page (page.id)}
              <text x={page.x} y={page.y + 110} fill="red">{probabilities[page.id].toFixed(2)}</text>
            {/each}
          {/if}
        </svg>
        {#each webpages as page (page.id)}
          <Webpage x={page.x-360} y={page.y} />
        {/each}
      </div>
      <div class="chart-two">
        <svg id="chart2" />
      </div>
    </div>
  </div>
</section>

<style>
  #chart1 {
    width: 100%;
    height: 100%;
    background-size: contain;
    background-repeat: no-repeat;
  }

  .chart-one {
    width: 70%;
    height: 70%;
    /* border: 3px solid skyblue; */
  }

  .charts-container {
    position: sticky;
    top: 10%;
    display: grid;
    width: 50%;
    height: 170vh;
  }

  .section-container {
    margin-top: 1em;
    text-align: center;
    transition: background 100ms;
    display: flex;
  }

  .step {
    height: 110vh;
    display: flex;
    place-items: center;
    justify-content: flex-end;
    padding-left: 40%;
  }

  .step-content {
    font-size: 18px;
    background: var(--bg);
    color: #ccc;
    border-radius: 20px; /* Added for rounded edges */
    padding: 0.5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    text-align: left;
    width: 75%;
    margin: auto;
    max-width: 500px;
    font-family: var(--font-main);
    line-height: 1.3;
    border: 5px solid var(--default);
  }

  .step.active .step-content {
    background: #f1f3f3ee;
    color: var(--squid-ink);
  }

  .steps-container {
    height: 100%;
  }

  .steps-container {
    flex: 1 1 40%;
    z-index: 10;
  }

  @media screen and (max-width: 950px) {
    .section-container {
      flex-direction: column-reverse;
    }
    .steps-container {
      pointer-events: none;
    }
    .charts-container {
      top: 7.5%;
      width: 95%;
      margin: auto;
    }
    .step {
      height: 130vh;
    }
    .step-content {
      width: 95%;
      max-width: 768px;
      font-size: 17px;
      line-height: 1.6;
    }
    .spacer {
      height: 100vh;
    }
  }

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
