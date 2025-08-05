<script>
    export let x;
    export let y;
    export let country;
    export let statusColors;

    import { fly, fade } from "svelte/transition";

    let tooltipWidth;
</script>

<div
    class="tooltip"
    in:fly={{ y: 10, duration: 120, delay: 120 }}
    out:fade
    style="position: absolute; top: {y}px; left: {x + 2}px;"
    bind:clientWidth={tooltipWidth}
>
    <p class="title">{country.country}</p>
    <div>
        <span style="background: black; height: 0.5px; width: 85%; position: absolute"/>
    </div>
    <p style="padding-top: 6px">{country.Status === 1 ? "Arancel acordado:" :
                country.Status === 3 ? "Arancel impuesto:" :
                country.Status === 2 ? "En negociaciones:" : 
                "Arancel mínimo:"}</p>
    <p style="font-style: italic; margin: 3px 0px">{country.Rate || "10"}%</p>
    <div>
        <span
            class="top-bar"
            style="background: {statusColors[country.Status]};
        width: {tooltipWidth}"
        />
    </div>
</div>

<style>
    .tooltip {
        background-color: white;
        box-shadow: 2px 3px 8px rgba(0, 0, 0, 0.15);
        padding: 8px 10px 10px;
        border-radius: 4px;
        pointer-events: none;
        width: max-content;
        max-width: 200px;
    }

    .top-bar {
        height: 4px;
        width: 100%;
        position: absolute;
        bottom: 0;
        left: 0;
    }

    .title {
        margin: 0 0 4px 0;
        font-size: 0.9rem;
        font-weight: 500;
    }

    p {
        margin: 0;
        font-size: 0.85rem;
        color: #333;
    }
</style>
