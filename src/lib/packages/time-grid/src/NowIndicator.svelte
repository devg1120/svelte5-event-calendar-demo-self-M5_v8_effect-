<script>
    import { getContext } from "svelte";

    let { slotDuration, slotHeight, theme, _now, _today, _slotTimeLimits } = getContext("state");

    let start = $state();
    let top = $state(0);

    $effect(() => {
        start = ($_now - $_today) / 1000 / 60;
    });
    $effect(() => {
        // Style
        let step = $slotDuration.seconds / 60;
        let offset = $_slotTimeLimits.min.seconds / 60;
        top = ((start - offset) / step) * $slotHeight;
    });
</script>

<div class={$theme.nowIndicator} style="top:{top}px"></div>
