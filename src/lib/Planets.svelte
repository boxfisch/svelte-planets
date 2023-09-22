<script lang="ts">
  import { extent, bisector } from "d3-array";
  import { scaleLinear, scaleSqrt } from "d3-scale";
  import { select } from "d3-selection";
  import { fade } from "svelte/transition";

  import data from "../assets/data.js";

  type dType = {
    size: number;
    name: string;
  };

  let height = 200;
  let width = 600;
  let bargap = 0;
  let showTooltip = false;
  let index: number | undefined = undefined;
  $: tooltipPos = index ? xScale(index) : 0;

  const padding = { top: 0, right: 50, bottom: 0, left: 50 };

  $: xExtent = [0, data.length];
  $: yExtent = extent(data.map((d: dType) => d.size)) as Iterable<number>;

  $: xScale = scaleLinear()
    .domain(xExtent)
    .range([padding.left, width - padding.right]);
  $: yScale = scaleSqrt().domain(yExtent).range([20, 60]);

  $: barWidth = xScale(1) - xScale(0) - bargap;

  function handleHover(e) {
    select(this).selectAll("circle").attr("cy", 0);
    select(this).selectAll("text").attr("fill", "white");

    index = e.target.id.split("-")?.[1];
    showTooltip = true;
  }

  function handleExit(e) {
    select(this)
      .selectAll("circle")
      .attr("cy", height / 2);
    select(this).selectAll("text").attr("fill", "gray");
    showTooltip = false;
  }
</script>

<div class="container">
  <div class="chart" bind:clientHeight={height} bind:clientWidth={width}>
    {#if showTooltip}
      <div
        class="tooltip"
        style:left="{tooltipPos}px"
        style:top="{height / 2}px"
        in:fade
      >
        Lorum ipsum dolor sit amit
      </div>
    {/if}

    <svg>
      {#each data as point, i}
        <g
          transform="translate({xScale(i)}, 0)"
          on:mouseenter={handleHover}
          on:mouseleave={handleExit}
          id="lane-{i}"
        >
          <rect y={padding.top} {height} x={-barWidth / 2} width={barWidth} />
          <circle
            r={yScale(point.size)}
            id="{i}-circle"
            cy={height / 2}
            fill={showTooltip && tooltipPos !== xScale(i)
              ? "transparent"
              : "#a11"}
            stroke={showTooltip && tooltipPos !== xScale(i)
              ? "#00000030"
              : "none"}
          />
          <text fill="grey" style:font-size=".8rem">{point.name}</text>
        </g>
      {/each}
    </svg>
  </div>
</div>

<style>
  .chart {
    max-width: 600px;
    margin-inline: auto;
    margin-block: 5rem;
  }

  svg {
    overflow: visible;
    height: 200px;
  }

  circle {
    transition: all 0.5s ease;
    opacity: 0.7;
    stroke-dasharray: 2;
  }

  rect {
    fill: transparent;
  }

  tspan,
  text {
    font-family: "Andale Mono";
    font-size: 0.7rem;
    text-anchor: middle;
    transition: all 0.5s ease;
    transition-delay: 500ms;
    dominant-baseline: middle;
  }

  .tooltip {
    position: absolute;
    transform: translateX(-50%);
    font-family: "Andale Mono";
    font-size: 0.6rem;
    color: #373737;
    opacity: 0.7;
    max-width: 100px;
    text-align: center;
    pointer-events: none;
  }
</style>
