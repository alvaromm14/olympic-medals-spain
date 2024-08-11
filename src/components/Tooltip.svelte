<script>
    export let data;
    export let tooltipX;
    export let tooltipY;
    export let width;
    export let height;
  
    import { fly, fade } from "svelte/transition";
  
    let tooltipWidth;
  
    const xNudge = 10;
    const yNudge = -40;
  
    $: xPosition = tooltipX + tooltipWidth + xNudge > width 
        ? (tooltipX - tooltipWidth - xNudge > 0 
            ? tooltipX - tooltipWidth - xNudge 
            : width - tooltipWidth)
        : tooltipX + xNudge;
    
    $: yPosition = tooltipY > height ? tooltipY - 115 : tooltipY + yNudge ;
  
  $: console.log(yPosition)
</script>

<div
  class="tooltip"
  in:fly={{ y: 10, duration: 120, delay: 120 }}
  out:fade
  style="position: absolute;
    top: {yPosition}px;
    left: {xPosition}px;"
  bind:clientWidth={tooltipWidth}
>
  <h1>{data.deporte}</h1>
  <h2>{data.nombre}</h2>
</div>

<style>
    .tooltip {
        background-color: white;
        box-shadow: 2px 3px 8px rgba(0, 0, 0, 0.15);
        padding: 8px 6px;
        border-radius: 3px;
        pointer-events: none;
        transition: top 50ms ease, left 50ms ease;
        max-width: 320px;
    }

    h1 {
        font-size: 0.9rem;
        font-weight: 600;
        margin: 4px 4px 4px 4px;
    }

    h2 {
        font-size: 0.9rem;
        font-weight: 400;
        font-style: italic;
        margin: 4px 4px 0px 4px;
    }
</style>