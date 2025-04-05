<script lang="ts">
   import { untrack } from "svelte";
    import { getContext } from "svelte";
    import Section from "./Section.svelte";
    interface Props {
        children?: import("svelte").Snippet;
    }

    let { children }: Props = $props();

    let { _events, _bodyEl, _viewDates, _slotTimeLimits, _times, scrollTime, slotDuration, slotHeight, theme } =
        getContext("state");

    let el = $state();
    let compact = $state();
    let lines_list = $state([]);

    function scrollToTime() {
        el.scrollTop =
            (($scrollTime.seconds - $_slotTimeLimits.min.seconds) / $slotDuration.seconds - 0.5) * $slotHeight;
    }
    $effect(() => {
        $_bodyEl = el;
    });


    $effect(() => {
        compact = $slotDuration.seconds >= 3600;
        lines_list.length = $_times.length;
        //console.log("lines_list.length",lines_list.length)
    });

    
    $effect(() => {
        if (el) {
            $_viewDates;
            $scrollTime;
            scrollToTime();
        }
    });
</script>

<div bind:this={el} class="{$theme.body}{compact ? ' ' + $theme.compact : ''}">
    <div class={$theme.content}>
        <Section>
            {#snippet lines()}
                {#each lines_list as line}
                    <div class={$theme.line}></div>
                {/each}
            {/snippet}
            {@render children?.()}
        </Section>
    </div>
</div>

<!--
<div bind:this={el} class="{$theme.body}{compact ? ' ' + $theme.compact : ''}">
    <div class={$theme.content}>
        <Section>
            {#snippet lines()}
                {#each lines as line}
                    <div class={$theme.line}></div>
                {/each}
            {/snippet}
            {@render children?.()}
        </Section>
    </div>
</div>
-->

<!--
<div
    bind:this={el}
    class="{$theme.body}{compact ? ' ' + $theme.compact : ''}"
>
    <div class="{$theme.content}">
        <Section>
            <svelte:fragment slot="lines">
                {#each lines as line}
                    <div class="{$theme.line}"></div>
                {/each}
            </svelte:fragment>
            <slot></slot>
        </Section>
    </div>
</div>
-->
