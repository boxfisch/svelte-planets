<script lang="ts">
  import { extent, bisector } from "d3-array";
  import { scaleLinear, scaleSqrt, scaleOrdinal } from "d3-scale";
  import { select } from "d3-selection";
  import { shuffle } from "d3-array";
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

  const colors = [
    "#e1edf8",
    "#cadef0",
    "#abcfe6",
    "#82badb",
    "#59a1cf",
    "#3787c0",
    "#1c6aaf",
    "#0b4d94",
  ];
  shuffle(colors);
  const colorScale = scaleOrdinal()
    .domain(data.map((d) => d.name))
    .range(colors);

  $: xExtent = [0, data.length];
  $: yExtent = extent(data.map((d: dType) => d.size)) as Iterable<number>;

  $: xScale = scaleLinear()
    .domain(xExtent)
    .range([padding.left, width - padding.right]);
  $: yScale = scaleSqrt().domain(yExtent).range([30, 70]);

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
            opacity=".8"
            cy={height / 2}
            fill={showTooltip && tooltipPos !== xScale(i)
              ? "transparent"
              : colorScale(point.name)}
            stroke={showTooltip && tooltipPos !== xScale(i)
              ? "#00000030"
              : "none"}
          />
          {#if !showTooltip && point.name === "Moon"}
            <circle
              r={yScale(point.size)}
              cy={height / 2 - 15}
              cx="15"
              fill="white"
              class="moonshadow"
              in:fade={{ delay: 400 }}
            />
          {/if}
          <text fill="grey" style:font-size=".8rem">{point.name}</text>
        </g>
      {/each}
    </svg>
  </div>
</div>

<style>
  .chart {
    max-width: 800px;
    margin-inline: auto;
    margin-block: 5rem;
  }

  svg {
    overflow: visible;
    height: 200px;
  }

  circle {
    transition: all 0.5s ease;
    stroke-dasharray: 2;
  }

  rect {
    fill: transparent;
  }

  tspan,
  text {
    font-family: "Andale Mono";
    font-size: 0.9rem;
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
