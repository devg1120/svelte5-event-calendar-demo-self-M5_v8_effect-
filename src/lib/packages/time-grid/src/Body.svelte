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
    let lines = $state([]);
    //let lines = $state(new Array(49));

    function scrollToTime() {
        el.scrollTop =
            (($scrollTime.seconds - $_slotTimeLimits.min.seconds) / $slotDuration.seconds - 0.5) * $slotHeight;
    }
    $effect(() => {
        $_bodyEl = el;
    });


    $effect(() => {
        compact = $slotDuration.seconds >= 3600;
        lines.length = $_times.length;
        console.log("lines.length",lines.length)
    });

    //let  lines = $state([$_times.length]);
    //let  lines = $state([49]);
    //let lines = $derived.by(() => { return new Array($_times.length);} )

    
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
                {#each lines as line}
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
