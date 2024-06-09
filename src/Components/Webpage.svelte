<script>
    export let x = 0;
    export let y = 0;
    export let transformOrigin = 'center center';
  </script>
  
  <style>
    .webpage {
      position: absolute;
      transition: transform 1s ease-out;
    }
  </style>
  
  <svg class="webpage" fill="#000000" width="100" height="100" style="transform-origin: {transformOrigin}; transform: translate({x}px, {y}px)">
      <path d="M52,64c2.206,0,4-1.794,4-4V4c0-2.206-1.794-4-4-4H26c-1.638,0-3.668,0.841-4.828,2L10,13.171C8.841,14.33,8,16.36,8,18v42
          c0,2.206,1.794,4,4,4H52z M52,4l0,56h0H12V18h10c2.206,0,4-1.794,4-4V4H52z M22,6.828V14h-7.172L22,6.828z"/>
      <path d="M32,16h12c1.104,0,2-0.896,2-2s-0.896-2-2-2H32c-1.104,0-2,0.896-2,2S30.896,16,32,16z"/>
      <path d="M44,22H20c-1.104,0-2,0.896-2,2s0.896,2,2,2h24c1.104,0,2-0.896,2-2S45.104,22,44,22z"/>
      <path d="M44,32H20c-1.104,0-2,0.896-2,2s0.896,2,2,2h24c1.104,0,2-0.896,2-2S45.104,32,44,32z"/>
      <path d="M44,42H20c-1.104,0-2,0.896-2,2s0.896,2,2,2h24c1.104,0,2-0.896,2-2S45.104,42,44,42z"/>
      <path d="M44,52H20c-1.104,0-2,0.896-2,2s0.896,2,2,2h24c1.104,0,2-0.896,2-2S45.104,52,44,52z"/>
  </svg>
  