<script lang="ts">
    import { untrack } from "svelte";

    import { getContext, onMount } from "svelte";
    import {
        ancestor,
        createEventClasses,
        createEventContent,
        height,
        max,
        toEventWithLocalDates,
        toViewWithLocalDates,
        setContent,
        repositionEvent,
        resourceBackgroundColor,
        resourceTextColor,
        helperEvent,
        keyEnter,
        task,
        rect,
        bgEvent,
        isFunction,
    } from "@event-calendar/core";

    interface Props {
        chunk: any;
        longChunks?: any;
        inPopup?: boolean;
        dates?: any;
    }

    let { chunk, longChunks = {}, inPopup = false, dates = [] }: Props = $props();

    let {
        dayMaxEvents,
        displayEventEnd,
        eventAllUpdated,
        eventBackgroundColor,
        eventTextColor,
        eventClick,
        eventColor,
        eventContent,
        eventClassNames,
        eventDidMount,
        eventMouseEnter,
        eventMouseLeave,
        resources,
        theme,
        _view,
        _intlEventTime,
        _interaction,
        _iClasses,
        _hiddenEvents,
        _popupDate,
        _tasks,
    } = getContext("state");

    let el = $state();
    let event = $state();
    let classes = $state();
    let style = $state();
    let content = $state();
    let timeText = $state();
    let margin = $state(1);
    let hidden = $state(false);
    let display = $state();
    let onclick = $derived(createHandler($eventClick, display));

    $effect.pre(() => {
        event = chunk.event;
    });

    $effect(() => {
        untrack(() => {
            display = event.display;

            // Class & Style
            let bgColor =
                event.backgroundColor ||
                resourceBackgroundColor(event, $resources) ||
                $eventBackgroundColor ||
                $eventColor;
            let txtColor = event.textColor || resourceTextColor(event, $resources) || $eventTextColor;

            if (event.allDay) {
                txtColor = "#000000";
            }

            if (bgEvent(display)) {
                style = `width:calc(${chunk.days * 100}% + ${chunk.days - 1}px);`;
                //style = `width:calc(${chunk.days * 100}% );`;
            } else {
                let marginTop = margin;
                if (event._margin) {
                    // Force margin for helper events
                    let [_margin, _dates] = event._margin;
                    if (chunk.date >= _dates[0] && chunk.date <= _dates.at(-1)) {
                        marginTop = _margin;
                    }
                }
                if (event.allDay) {
                  //console.log(chunk.event.title)
                  //console.log(chunk.prev_week_continue)
                  //console.log(chunk.next_week_continue)
                  style = `width:calc(${chunk.days * 100}% + ${(chunk.days - 1) * 2}px);` + `margin-top:${marginTop}px;`;
                  if ( !chunk.prev_week_continue ) {
                     style += "   border-top-left-radius: 20px;"
                     style += "   border-bottom-left-radius: 20px;"
                  }
                  if ( !chunk.next_week_continue ) {
                     style += "   border-top-right-radius: 20px;"
                     style += "   border-bottom-right-radius: 20px;"
                  }
                     style += " box-shadow: 0 4px 0 #808080; ";


                } else {
                  //style = `width:calc(${chunk.days * 100}% + ${(chunk.days - 1) * 7}px);` + `margin-top:${marginTop}px;`;
                  if ( chunk.next_week_continue ) {
                     style = `width:calc(${chunk.days * 100}% + ${(chunk.days - 1) * 7 + 7}px);` + `margin-top:${marginTop}px;`;
                     style += "border-right:  thick double blue;"
                  } else {
                     style = `width:calc(${chunk.days * 100}% + ${(chunk.days - 1) * 7}px);` + `margin-top:${marginTop}px;`;
                  }
                  if ( chunk.prev_week_continue ) {
                     style += "border-left:  thick double blue;"
                  }
                }
            }
            if (bgColor) {
                style += `background-color:${bgColor};`;
            }
            if (txtColor) {
                style += `color:${txtColor};`;
            }
/*
            if (event.allDay) {
                //style += "border-top: solid red;";
                style += " box-shadow: 0 5px 0 #ca1c30; ";
            }
*/
            if (hidden) {
                style += "visibility:hidden;";
            }
            style += event.styles.join(";");

            classes = [
                bgEvent(display) ? $theme.bgEvent : $theme.event,
                ...$_iClasses([], event),
                ...createEventClasses($eventClassNames, event, $_view),
            ].join(" ");
        });
    });

    // Content
    $effect(() => {
        [timeText, content] = createEventContent(
            chunk,
            $displayEventEnd,
            $eventContent,
            $theme,
            $_intlEventTime,
            $_view,
        );
    });

    onMount(() => {
        if (isFunction($eventDidMount)) {
            $eventDidMount({
                event: toEventWithLocalDates(event),
                timeText,
                el,
                view: toViewWithLocalDates($_view),
            });
        }
    });

    $effect(() => {
        if (isFunction($eventAllUpdated) && !helperEvent(display)) {
            task(() => $eventAllUpdated({ view: toViewWithLocalDates($_view) }), "eau", _tasks);
        }
    });

    function createHandler(fn, display) {
        //const createHandler = (fn, display) => {
        return !helperEvent(display) && isFunction(fn)
            ? (jsEvent) => fn({ event: toEventWithLocalDates(event), el, jsEvent, view: toViewWithLocalDates($_view) })
            : undefined;
    }

    function createDragHandler(interaction, resize) {
        return interaction.action
            ? (jsEvent) =>
                  $_interaction.action.drag(
                      event,
                      jsEvent,
                      resize,
                      inPopup ? $_popupDate : null,
                      [rect(el).top - rect(ancestor(el, 1)).top, dates],
                      chunk.zeroDuration,
                  )
            : undefined;
    }

    export function reposition() {
        if (!el) {
            return;
        }
        margin = repositionEvent(chunk, longChunks, height(el));
        if ($dayMaxEvents === true) {
            hide();
        } else {
            hidden = false;
        }
    }

    function hide() {
        let dayEl = ancestor(el, 2);
        let h = height(dayEl) - height(dayEl.firstElementChild) - footHeight(dayEl);
        hidden = chunk.bottom > h;
        let update = false;
        // Hide or show the event throughout all days
        for (let date of chunk.dates) {
            let hiddenEvents = $_hiddenEvents[date.getTime()];
            if (hiddenEvents) {
                let size = hiddenEvents.size;
                if (hidden) {
                    hiddenEvents.add(chunk.event);
                } else {
                    hiddenEvents.delete(chunk.event);
                }
                if (size !== hiddenEvents.size) {
                    update = true;
                }
            }
        }
        if (update) {
            $_hiddenEvents = $_hiddenEvents;
        }
    }

    function footHeight(dayEl) {
        let h = 0;
        for (let i = 0; i < chunk.days; ++i) {
            h = max(h, height(dayEl.lastElementChild));
            dayEl = dayEl.nextElementSibling;
            if (!dayEl) {
                break;
            }
        }
        return h;
    }

    // Onclick handler

    const SvelteComponent = $derived($_interaction.resizer);
</script>

<!-- svelte-ignore a11y_no_noninteractive_tabindex -->
<article
    bind:this={el}
    class={classes}
    {style}
    role={onclick ? "button" : undefined}
    tabindex={onclick ? 0 : undefined}
    onclick={onclick || undefined}
    onkeydown={onclick && keyEnter(onclick)}
    onmouseenter={() => createHandler($eventMouseEnter, display)}
    onmouseleave={() => createHandler($eventMouseLeave, display)}
    onpointerdown={!helperEvent(display) && createDragHandler($_interaction)}
>
    <SvelteComponent start {event} on:pointerdown={createDragHandler($_interaction, ["x", "start"])} />
    <div class={$theme.eventBody} use:setContent={content}></div>
    <SvelteComponent {event} on:pointerdown={createDragHandler($_interaction, ["x", "end"])} />
</article>
