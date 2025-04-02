<script lang="ts">
    import { untrack } from "svelte";

    import { getContext, tick } from "svelte";
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

    let { dates } = $props();

    let {
        _events,
        _iEvents,
        _queue2,
        _hiddenEvents,
        resources,
        filterEventsWithResources,
        hiddenDays,
        theme,
        validRange,
    } = getContext("state");

    //let chunks = $state(),
    //    bgChunks = $state(),
    //    longChunks = $state(),
    let  iChunks = $state([]);

    let start = $state();
    let end = $state();
    let refs = $state([]);
    let debounceHandle = {};
    //let allDaySlotHeight = 100;
    //let start = $state(limitToRange(dates[0], $validRange));
    //let  end =  $state(addDay(cloneDate(limitToRange(dates.at(-1), $validRange))));
     //console.log(start)
     //console.log(end)
/*
    let  start = $derived.by(() => {
            return limitToRange(dates[0], $validRange);

    })
    let  end  = $derived.by(() => {
            return addDay(cloneDate(limitToRange(dates.at(-1), $validRange)));

    })
*/

    let { chunks, bgChunks , longChunks, week_array, bg_week_array, allDaySlotHeight} = $derived.by(() => {

            let chunks = [];
            let bgChunks = [];
            for (let event of $_events) {
                if (eventIntersects(event, start, end, $filterEventsWithResources ? $resources : undefined)) {
                    let chunk = createEventChunk(event, start, end);
                    //if (bgEvent(event.display)) {
                        if (event.allDay) {
			    //console.log("bgChunk:", chunk)
                            bgChunks.push(chunk);
                     //   }
                    } else {
                        chunks.push(chunk);
                    }
                }
            }
            let tmp = prepareEventChunks(bgChunks, $hiddenDays);
            let longChunks = prepareEventChunks(chunks, $hiddenDays);
            // Run reposition only when events get changed
            //reposition();

           //console.log( "chanks",chunks  );
           //console.log( "bgChanks", bgChunks );
           //console.log( "longChunks", longChunks );

           //--------------------------------------------
           // week_array

           let event_max = chunks.length
           let week_array = [];
           for(var i=0; i<7; i++) {
             week_array.push(Array(event_max).fill(0));
           }

           if (chunks.length > 0) { 
                 //console.log(week_array)
                   //console.log(chunks.length)
                   let event_no = 0
                   for ( let chunk of chunks) {
                        event_no++;
                        //console.log("chunk.start", chunk.start)
                        //console.log("chunk.event.title", chunk.event.title)
                        //console.log("chunk.days", chunk.days)
                        let diffMilliSec = chunk.start -start;
                        let week_index = parseInt(diffMilliSec / 1000 / 60 / 60 / 24);
                        //console.log("week_index:",week_index, chunk.days)
                        for ( var c=0; c< chunks.length ; c++) {
                            if (week_array[week_index][c]== 0) {
                                week_array[week_index][c] = event_no;
                                for ( var s=1; s< chunk.days ; s++) {
                                 //week_array[week_index+s][c] = event_no;
                                 week_array[week_index+s][c] = -1;
                                }
                                break
                            } 
                            

                        }

                   }
                 for(var i=0; i<7; i++) {
                     for (let x = week_array[i].length -1 ; x => 0; x--) {
                          //console.log(week_array[i][x])
                          if (week_array[i][x] != 0) {
                            //week_array[i][x+1] = -3
                            break
                          }
                            week_array[i][x] = -3
                     }
                 }
                 //console.log(week_array)
           }

           //--------------------------------------------
           // bg_week_array

           let bg_event_max = bgChunks.length
           let bg_week_array = [];
           for(var i=0; i<7; i++) {
             bg_week_array.push(Array(bg_event_max).fill(0));
           }

           if (bgChunks.length > 0) { 
                 //console.log(week_array)
                   //console.log(chunks.length)
                   let event_no = 0
                   for ( let chunk of bgChunks) {
                        event_no++;
                        //console.log("chunk.start", chunk.start)
                        //console.log("chunk.event.title", chunk.event.title)
                        //console.log("chunk.days", chunk.days)

                        let diffMilliSec = chunk.start -start;
                        let week_index = parseInt(diffMilliSec / 1000 / 60 / 60 / 24);
                        //console.log("week_index:",week_index, chunk.days)
                        for ( var c=0; c< bgChunks.length ; c++) {
                            if (bg_week_array[week_index][c]== 0) {
                                bg_week_array[week_index][c] = event_no;
                                for ( var s=1; s< chunk.days ; s++) {
                                 //week_array[week_index+s][c] = event_no;
                                 bg_week_array[week_index+s][c] = -1;
                                }
                                break
                            } 
                            

                        }

                   }
                 for(var i=0; i<7; i++) {
                     for (let x = bg_week_array[i].length -1 ; x => 0; x--) {
                          //console.log(bg_week_array[i][x])
                          if (bg_week_array[i][x] != 0) {
                            //bg_week_array[i][x+1] = -3
                            break
                          }
                            bg_week_array[i][x] = -3
                     }
                 }
                 //console.log(bg_week_array)
           }
                 let slotn = 0
                 for(var i=0; i<7; i++) {
                     for (let x = 0;  x < bg_week_array[i].length ;x++) {
                          if (bg_week_array[i][x] != -3) {
                                   if (slotn < x + 1) {
                                        slotn = x + 1
                                   }
                          }
                     }
                 }
                 //console.log("slotn:", slotn)
                let allDaySlotHeight = 30 * slotn;

            //console.log(start)
            //console.log(end)
            //console.log(chunks)
           //console.log( "dates",dates  );
           //console.log( "start",start  );
           //console.log( "chanks.length",chunks.length  );
           //console.log( "chanks",chunks  );
           //console.log( week_array  );

           return { chunks, bgChunks, longChunks , week_array, bg_week_array, allDaySlotHeight};


    })
/*
    let allDaySlotHeight = 100;
    let slotn  = $derived.by(() => {
                 let slotn = 0
                 for(var i=0; i<7; i++) {
                     for (let x = 0;  x < bg_week_array[i].length ;x++) {
                          if (bg_week_array[i][x] == -3) {
                                   if (slotn < x -1) {
                                        slotn = x -1
                                   }
                          }
                     }
                 }
                 console.log("slotn:", slotn)
                return slotn
     })
*/
    _events.subscribe((v) => {
        chunks = [];
        bgChunks = [];
        for (let event of $_events) {
            if (eventIntersects(event, start, end, $filterEventsWithResources ? $resources : undefined)) {
                let chunk = createEventChunk(event, start, end);
                if (bgEvent(event.display)) {
                    if (event.allDay) {
                        bgChunks.push(chunk);
                    }
                } else {
                    chunks.push(chunk);
                }
            }
        }
        prepareEventChunks(bgChunks, $hiddenDays);
        longChunks = prepareEventChunks(chunks, $hiddenDays);
        reposition();
    });

    $effect.pre(() => {
        //untrack(() => {
            start = limitToRange(dates[0], $validRange);
            end = addDay(cloneDate(limitToRange(dates.at(-1), $validRange)));
           //console.log(start)
        //});
    });



    //let debounceHandle = {};
    function reposition() {
        debounce(() => runReposition(refs, dates), debounceHandle, _queue2);
    }

/*
    $effect(() => {
        untrack(() => {
            chunks = [];
            bgChunks = [];
            for (let event of $_events) {
                if (eventIntersects(event, start, end, $filterEventsWithResources ? $resources : undefined)) {
                    let chunk = createEventChunk(event, start, end);
                    if (bgEvent(event.display)) {
                        if (event.allDay) {
                            bgChunks.push(chunk);
                        }
                    } else {
                        chunks.push(chunk);
                    }
                }
            }
            prepareEventChunks(bgChunks, $hiddenDays);
            longChunks = prepareEventChunks(chunks, $hiddenDays);
            // Run reposition only when events get changed
            reposition();
        });
    });
*/

    $effect(() => {
        iChunks = $_iEvents.map((event) => {
            let chunk;
            if (event && eventIntersects(event, start, end)) {
                chunk = createEventChunk(event, start, end);
                prepareEventChunks([chunk], $hiddenDays);
            } else {
                chunk = null;
            }
            return chunk;
        });
    });

    $effect(() => {
        untrack(() => {
            if ($_hiddenEvents) {
                // Schedule reposition during next update
                tick().then(reposition);
            }
        });
    });

    let weekRowHeight = 200;
</script>

<!--
<style>
      .weekRow { height:var(--weekRowHeight)}
</style>
  class="{$theme.days} weekRow" role="row"
  style="--weekRowHeight: {weekRowHeight}px"
-->

<div 
  class="{$theme.days} " role="row"
>
    {#key chunks}
        {#each dates as date, i}
            <Day {date} {chunks} {bgChunks} {longChunks} {iChunks} {dates} {week_array} {bg_week_array} {i}  {allDaySlotHeight} bind:this={refs[i]} />
        {/each}
    {/key}
</div>

<svelte:window onresize={reposition} />
