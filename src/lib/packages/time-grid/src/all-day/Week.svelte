<script lang="ts">
    import { getContext } from "svelte";
    import {
        addDay,
        bgEvent,
        cloneDate,
        createEventChunk,
        debounce,
        eventIntersects,
        limitToRange,
        prepareEventChunks,
        runReposition,
    } from "@event-calendar/core";
    import Day from "./Day.svelte";

    let { dates, resource = undefined } = $props();

    let { _events, _iEvents, _queue2, hiddenDays, resources, filterEventsWithResources, validRange } =
        getContext("state");

    let iChunks = $state([]);

    let start = $state();
    let end = $state();
    let refs = $state([]);
    let resourceFilter = $state();

    let { chunks, bgChunks, longChunks, week_array, allDaySlotHeight } = $derived.by(() => {
        let chunks = [];
        let bgChunks = [];
        for (let event of $_events) {
            if (event.allDay && eventIntersects(event, start, end, resourceFilter)) {
                let chunk = createEventChunk(event, start, end);
                if (bgEvent(event.display)) {
                    bgChunks.push(chunk);
                } else {
                    chunks.push(chunk);
                }
            }
        }
        prepareEventChunks(bgChunks, $hiddenDays);
        let longChunks = prepareEventChunks(chunks, $hiddenDays);
        // Run reposition only when events get changed
        //reposition();                                                      //BUG

        console.log("bgChunks", bgChunks);
        console.log("longChunks", longChunks);
        //--------------------------------------------
        // allDay_week_array

        let event_max = chunks.length;
        let week_array = [];
        for (var i = 0; i < 7; i++) {
            week_array.push(Array(event_max).fill(0));
        }

        if (chunks.length > 0) {
            let event_no = 0;
            for (let chunk of chunks) {
                event_no++;

                let diffMilliSec = chunk.start - start;
                let week_index = parseInt(diffMilliSec / 1000 / 60 / 60 / 24);
                //console.log("week_index:",week_index, chunk.days)
                for (var c = 0; c < chunks.length; c++) {
                    if (week_array[week_index][c] == 0) {
                        week_array[week_index][c] = event_no;
                        for (var s = 1; s < chunk.days; s++) {
                            //week_array[week_index+s][c] = event_no;
                            week_array[week_index + s][c] = -1;
                        }
                        break;
                    }
                }
            }
            for (var i = 0; i < 7; i++) {
                for (let x = week_array[i].length - 1; (x) => 0; x--) {
                    if (week_array[i][x] != 0) {
                        break;
                    }
                    week_array[i][x] = -3;
                }
            }
            //console.log(week_array)
        }
        let slotn = 0;
        for (var i = 0; i < 7; i++) {
            for (let x = 0; x < week_array[i].length; x++) {
                if (week_array[i][x] != -3) {
                    if (slotn < x + 1) {
                        slotn = x + 1;
                    }
                }
            }
        }
        //console.log("slotn:", slotn)
        let allDaySlotHeight = 30 * slotn;

        console.log("week_array", week_array);

        return { chunks, bgChunks, longChunks, week_array, allDaySlotHeight };
    });

    _events.subscribe((v) => {
        //console.log("Week All-days event Update")

        chunks = [];
        bgChunks = [];
        for (let event of $_events) {
            if (event.allDay && eventIntersects(event, start, end, resourceFilter)) {
                let chunk = createEventChunk(event, start, end);
                if (bgEvent(event.display)) {
                    bgChunks.push(chunk);
                } else {
                    chunks.push(chunk);
                }
            }
        }

        prepareEventChunks(bgChunks, $hiddenDays);
        longChunks = prepareEventChunks(chunks, $hiddenDays);
        // Run reposition only when events get changed
        //reposition();
    });

    $effect.pre(() => {
        start = limitToRange(dates[0], $validRange);
        end = addDay(cloneDate(limitToRange(dates.at(-1), $validRange)));
    });

    $effect.pre(() => {
        resourceFilter = resource ?? ($filterEventsWithResources ? $resources : undefined);
    });

    let debounceHandle = {};
    function reposition() {
        debounce(() => runReposition(refs, dates), debounceHandle, _queue2);
    }

    $effect(() => {
        iChunks = $_iEvents.map((event) => {
            let chunk;
            if (event && event.allDay && eventIntersects(event, start, end, resource)) {
                chunk = createEventChunk(event, start, end);
                prepareEventChunks([chunk], $hiddenDays);
            } else {
                chunk = null;
            }
            return chunk;
        });
    });
</script>

{#key longChunks}
    {#each dates as date, i}
        <!--
        <Day {date} {chunks} {bgChunks} {longChunks} {iChunks} {resource} bind:this={refs[i]} />
-->
        <Day {date} {chunks} {bgChunks} {longChunks} {week_array} {i} {iChunks} {resource} bind:this={refs[i]} />
    {/each}
{/key}

<svelte:window onresize={reposition} />
