<script>
    export let colorScale;
    export let data;

    import { format } from "d3-format";

    const minColor = colorScale.range()[0];
    const maxColor = colorScale.range()[1];
    const minValue = colorScale.domain()[0];
    const maxValue = colorScale.domain()[1];

    const suffixFormat = (d) => format(".2s")(d).replace("G", "B");

    $: percentOfMax = (data?.population / maxValue) * 100;

    import { fade } from "svelte/transition";

</script>

<div class="legend">
    <span class="label">{minValue}</span>
    <div class="bar" style:background="Linear-gradient(to right, {minColor} 0%, {maxColor} 100%)">
        {#if percentOfMax}
    <span transition:fade class="line" style:left={percentOfMax + "%"} />
{/if}
    </div>
    <span class="label">{suffixFormat(maxValue)}</span>
</div>

<style>
    
    .legend {
        display: flex;
        flex-direction: row;
        gap: 6px;
    }

    .bar {
        height: 15px;
        width: 100%;
        background: white;
        position: relative;
    }

    .label {
        color: white;
        font-size: 0.85rem;
        user-select: none;
    }

    .line {
        position: absolute;
        top: 0;
        height: 15px;
        width: 2px;
        background: white;
        transition: left 500ms cubic-bezier(1, 0, 0, 1);
    }
</style>