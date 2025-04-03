<script lang="ts">
    import { height } from "@event-calendar/core";

    //console.log("space:", height());
    let spaceHeight = 26;
</script>

<div class="eventSpace" style="--SpaceHeight: {spaceHeight}px"></div>

<style>
    .eventSpace {
        height: var(--SpaceHeight);
    }
</style>
