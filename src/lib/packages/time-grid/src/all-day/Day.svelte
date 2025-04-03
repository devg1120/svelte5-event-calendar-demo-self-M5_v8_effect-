<script lang="ts">
    import { getContext } from "svelte";
    import { datesEqual, outsideRange, runReposition, setPayload } from "@event-calendar/core";
    import Event from "./Event.svelte";
    import EventSpace from "./EventSpace.svelte";

    let { date, chunks, bgChunks, longChunks, week_array, i, iChunks = [], resource = undefined } = $props();

    let index = i;

    let { highlightedDates, theme, validRange, _interaction, _today } = getContext("state");

    let el = $state();
    //   let dayChunks = $derived(chunks.filter((chunk) => datesEqual(chunk.date, date)));

    let dayChunks = $derived.by(() => {
        let dayChunks = [];

        for (let no of week_array[index]) {
            //console.log(no)
            if (no == 0 || no == -1) {
                //console.log("skip")
                let chunk2 = JSON.parse(JSON.stringify(chunks[0]));
                chunk2.event.title = "SPACE";
                chunk2.space = true;
                dayChunks.push(chunk2);
            } else if (no == -3) {
                break;
            } else {
                //console.log("draw", no)
                let chunk = chunks[no - 1];
                //console.log(chunk)
                chunk.space = false;
                dayChunks.push(chunk);
            }
        }
        return dayChunks;
    });

    let dayBgChunks = $derived(bgChunks.filter((bgChunk) => datesEqual(bgChunk.date, date)));
    let isToday = $derived(datesEqual(date, $_today)),
        highlight = $derived($highlightedDates.some((d) => datesEqual(d, date))),
        disabled = $state();
    let refs = $state([]);

    $effect(() => {
        disabled = outsideRange(date, $validRange);
    });

    // dateFromPoint
    $effect(() => {
        if (el) {
            setPayload(el, () => ({
                allDay: true,
                date,
                resource,
                dayEl: el,
                disabled,
            }));
        }
    });

    export function reposition() {
        if (!disabled) {
            runReposition(refs, dayChunks);
        }
    }
</script>

<div
    bind:this={el}
    class="{$theme.day} {$theme.weekdays?.[date.getUTCDay()]}{isToday ? ' ' + $theme.today : ''}{highlight
        ? ' ' + $theme.highlight
        : ''}{disabled ? ' ' + $theme.disabled : ''}"
    role="cell"
    onpointerdown={!disabled ? $_interaction.action?.select : undefined}
>
    <div class={$theme.bgEvents}>
        {#if !disabled}
            {#each dayBgChunks as chunk (chunk.event)}
                <Event {chunk} />
            {/each}
        {/if}
    </div>
    <!-- Drag, Resize & Select -->
    {#if iChunks[0] && datesEqual(iChunks[0].date, date) && !disabled}
        <div class="{$theme.events} {$theme.preview}">
            <Event chunk={iChunks[0]} />
        </div>
    {/if}
    <div class={$theme.events}>
        {#if !disabled}
            {#each dayChunks as chunk, i (chunk.event)}
                <!--
                <Event {chunk} {longChunks} bind:this={refs[i]} />
-->
                {#if !chunk.space}
                    <Event {chunk} {longChunks} bind:this={refs[i]} />
                {:else}
                    <EventSpace />
                {/if}
            {/each}
        {/if}
    </div>
</div>
