<script>
  import world from "$data/110m.json";
  import data from "$data/data.json";
  import Tooltip from "$components/Tooltip.svelte";

  import * as topojson from "topojson-client";
  import { geoNaturalEarth1, geoPath, geoCentroid } from "d3-geo";
  import { scaleLinear } from "d3-scale";
  import { max } from "d3-array";
  import countriesEs from "i18n-iso-countries/langs/es.json";
  import i18nIsoCountries from "i18n-iso-countries";

  i18nIsoCountries.registerLocale(countriesEs);

  import { fade } from "svelte/transition";

  let countries = topojson
    .feature(world, world.objects.countries)
    .features.filter((country) => country.id !== "010");
  let borders = topojson.mesh(
    world,
    world.objects.countries,
    (a, b) => a !== b,
  );

  countries.forEach((country) => {
    const metadata = data?.find((d) => d.id === country.id);
    if (metadata) {
      country.Status = metadata.Status;
      country.Rate = metadata.Rate;
      country.country = metadata.country;
    } else {
      country.Status = null;
      country.country = i18nIsoCountries.getName(country.id, "es");
    }
  });

  let width = 400;
  $: height = (width > 800 ? width / 2.4 : width / 2.2)

  $: projection = geoNaturalEarth1()
    .scale(
      width < 480 ? width / 4.5 :
      width < 768 ? width / 5.2 :
      width / 6
    )
    .translate([width / 2, height / 1.7]);

  $: path = geoPath(projection);

  const statusColors = {
    "1": "#4ea3de", // (Preliminary deal)
    "3": "#ffa759", // (Tariff announced)
    "2": "#f7e074", // (In negotiations)
    null: "#e6e6e6", // sin estado
  };

  function getColor(status, id) {
    if (id === "840") return "#b3b3b3"; // gris más oscuro para EE. UU.
    return statusColors[status] || statusColors["null"];
  }

  const legendItems = [
    { label: "Arancel acordado", color: statusColors["1"] },
    { label: "Arancel impuesto", color: statusColors["3"] },
    { label: "En negociaciones", color: statusColors["2"] },
    { label: "Arancel base", color: statusColors[null] },
  ];

  let tooltipData;
  let tooltipX = 0;
  let tooltipY = 0;
  const countriesToLabel = ["124", "156", "484", "040", "826", "392", "356", "076"];

  window.addEventListener("DOMContentLoaded", (event) => {
    function updateIframeHeight() {
      const el = document.documentElement;
      const rect = el.getBoundingClientRect();
      const styles = window.getComputedStyle(el);
      const margin =
        parseFloat(styles.marginTop) + parseFloat(styles.marginBottom);
      const height = Math.ceil(rect.height + margin);

      window.parent.postMessage(
        {
          type: "resize-iframe",
          value: height,
        },
        "*",
      );
    }
    updateIframeHeight();

    if (window.ResizeObserver) {
      new ResizeObserver(() => {
        updateIframeHeight();
      }).observe(document.documentElement);
    } else {
      window.addEventListener("load", updateIframeHeight);
      window.addEventListener("resize", updateIframeHeight);
    }

    window.addEventListener(
      "message",
      (event) => {
        if (event.data.type === "request-resize") {
          updateIframeHeight();
        }
      },
      false,
    );
  });
</script>

<div class="chart-container" bind:clientWidth={width}>
  <div class="legend">
    {#each legendItems as item}
      <div class="legend-item">
        <div class="legend-color" style="background-color: {item.color}"></div>
        <span>{item.label}</span>
      </div>
    {/each}
  </div>
  <svg {width} {height}>
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    {#each countries as country}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <path
        d={path(country)}
        fill={getColor(country.Status, country.id)}
        stroke="none"
        on:mouseenter={() => {
          if (country.id !== "840") {
            tooltipData = country;
            tooltipX = event.clientX;
            tooltipY = event.clientY;
          }
        }}
        on:mousemove={(event) => {
          if (country.id !== "840" && tooltipData?.id === country.id) {
            tooltipX = event.clientX;
            tooltipY = event.clientY;
          }
        }}
        on:mouseleave={() => {
          if (country.id !== "840") {
            tooltipData = null;
          }
        }}
      />
    {/each}
    <path
      d={path(borders)}
      fill="none"
      stroke="white"
      stroke-width="0.5"
      user-select="none"
      pointer-events="none"
    />
    {#if Tooltip}
      <path
        d={path(tooltipData)}
        fill="transparent"
        stroke="black"
        stroke-width="1"
        user-select="none"
        pointer-events="none"
      />
    {/if}
    {#each countries as country}
      {#if countriesToLabel.includes(country.id) && !tooltipData && width > 768}
        {#if projection}
          <text
            in:fade={{ duration: 150 }}
            out:fade={{ duration: 150 }}
            x={projection(geoCentroid(country))[0]}
            y={projection(geoCentroid(country))[1]}
            text-anchor="middle"
            alignment-baseline="middle"
            font-size="0.8rem"
            font-weight="500"
            stroke="#fff"
            stroke-width="3"
            paint-order="stroke"
            user-select="none"
            pointer-events="none"
          >
            {country.country}
          </text>
        {/if}
      {/if}
    {/each}
  </svg>
  {#if tooltipData}
    <Tooltip x={tooltipX} y={tooltipY} {statusColors} country={tooltipData} />
  {/if}
  <div class="credits">
  <img
    src="https://elordenmundial.com/wp-content/uploads/2025/05/EOM-logo.png"
    alt="Logo EOM"
    class="eom-logo" />
</div>
</div>


<style>
  .chart-container {
    margin: 0 auto;
  }

  svg {
    overflow: visible;
  }

  .legend {
    display: flex;
    justify-content: center;
    margin-bottom: 1rem;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.9rem;
    user-select: none;
  }

  .legend-color {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    flex-shrink: 0;
  }

  .credits {
  display: flex;
  justify-content: space-between;
  align-items: end;
  margin-top: 12px;
}

.eom-logo {
  width: 80px;
  height: auto;
  margin-left: auto;
}

@media (max-width: 850px) {
  .eom-logo {
    width: 60px;
    margin-left: auto; 
  }
}

@media (max-width: 500px) {
  .eom-logo {
    width: 40px;
    margin-left: auto; 
  }
}
</style>
