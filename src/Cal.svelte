<script lang="ts">
    import Calendar from "@event-calendar/core";
    import TimeGrid from "@event-calendar/time-grid";
    import DayGrid from "@event-calendar/day-grid";
    import List from "@event-calendar/list";
    import ResourceTimeGrid from "@event-calendar/resource-time-grid";
    import ResourceTimeline from "@event-calendar/resource-timeline";
    import Interaction from "@event-calendar/interaction";

    let plugins = [TimeGrid, DayGrid, List, ResourceTimeGrid, ResourceTimeline, Interaction];
    //let plugins = [TimeGrid, DayGrid,  ResourceTimeGrid, ResourceTimeline, Interaction];
    let options: Calendar.Options = $state({
        //let options: Calendar.Options = {
        //view: "dayGridMonth",
        view: "timeGridWeek",
        //view: 'timeGridDay',
        headerToolbar: {
            start: "prev,next today",
            center: "title",
            end: "dayGridMonth,timeGridWeek,timeGridDay,listWeek resourceTimeGridWeek,resourceTimelineWeek",
        },
        resources: [
            { id: 1, title: "Resource A" },
            { id: 2, title: "Resource B" },
        ],
        allDaySlot: false,
        scrollTime: "09:00:00",
        events: createEvents(),
        views: {
            timeGridWeek: { pointer: true },
            resourceTimeGridWeek: { pointer: true },
            resourceTimelineWeek: {
                pointer: true,
                slotMinTime: "09:00",
                slotMaxTime: "21:00",
                slotWidth: 80,
                resources: [
                    { id: 1, title: "Resource A" },
                    { id: 2, title: "Resource B" },
                    { id: 3, title: "Resource C" },
                    { id: 4, title: "Resource D" },
                    { id: 5, title: "Resource E" },
                    { id: 6, title: "Resource F" },
                    { id: 7, title: "Resource G" },
                    { id: 8, title: "Resource H" },
                    {
                        id: 9,
                        title: "Resource I",
                        children: [
                            { id: 10, title: "Resource J" },
                            { id: 11, title: "Resource K" },
                            {
                                id: 12,
                                title: "Resource L",
                                children: [
                                    { id: 13, title: "Resource M" },
                                    { id: 14, title: "Resource N" },
                                    { id: 15, title: "Resource O" },
                                ],
                            },
                        ],
                    },
                ],
            },
        },
        dayMaxEvents: true,
        nowIndicator: true,
        selectable: true,
        editable: true,
    });
    //};

    function toggleAllDay() {
        options.allDaySlot = !options.allDaySlot;
    }

    function createEvents() {
        let days = [];
        //for (let i = -7; i < 7; ++i) {
        for (let i = -7; i < 14; ++i) {
            let day = new Date();
            let diff = i - day.getDay();
            day.setDate(day.getDate() + diff);
            days[i] = day.getFullYear() + "-" + _pad(day.getMonth() + 1) + "-" + _pad(day.getDate());
        }

        return [
            /*

            {
                start: days[-7] + " 10:00",
                end: days[-7] + " 14:00",
                resourceId: 1,
                title: "The calendar can display background and regular events",
                color: "#FE6B64",
            },
    
            {
                start: days[-7] + " 10:00",
                end: days[-7] + " 14:00",
                resourceId: 1,
                title: "The calendar can display background and regular events",
                color: "#4E6B00",
            },
    
            {
                start: days[-7] + " 10:00",
                end: days[-5] + " 14:00",
                resourceId: 1,
                title: "Long 1",
                color: "#ADD8E6",
            },
	    
    
            {
                start: days[-5] + " 10:00",
                end: days[-4] + " 14:00",
                resourceId: 1,
                title: "Long 2",
                color: "#AD00E6",
            },
	    
            {
                start: days[-6] + " 10:00",
                end: days[-4] + " 14:00",
                resourceId: 1,
                title: "allday 1",
                color: "#FFB6C1",
                allDay: true,
            },

            {
                start: days[-4] + " 10:00",
                end: days[-2] + " 14:00",
                resourceId: 1,
                title: "allday 2",
                color: "#DDA0DD",
                allDay: true,
            },

            {
                start: days[-5] + " 9:00",
                end: days[-5] + " 10:00",
                resourceId: 1,
                title: "END",
                color: "#00BFFF	",
            },
            {
                start: days[-5] + " 8:00",
                end: days[-5] + " 10:00",
                resourceId: 1,
                title: "The calendar can display background and regular events",
                color: "#00BFFF	",
            },
*/
            /*
            { start: days[0] + " 00:00", end: days[0] + " 09:00", resourceId: 1, display: "background" },
            { start: days[1] + " 12:00", end: days[1] + " 14:00", resourceId: 2, display: "background" },
            { start: days[2] + " 17:00", end: days[2] + " 24:00", resourceId: 1, display: "background" },
*/
            {
                start: days[0] + " 10:00",
                end: days[0] + " 14:00",
                resourceId: 1,
                title: "The calendar can display background and regular events",
                color: "#FE6B64",
            },
            {
                start: days[1] + " 16:00",
                end: days[2] + " 08:00",
                resourceId: 2,
                title: "An event may span to another day",
                color: "#B29DD9",
            },
            {
                start: days[2] + " 09:00",
                end: days[2] + " 13:00",
                resourceId: 2,
                title: "END",
                color: "#779ECB",
            },
            { start: days[3] + " 14:00", end: days[3] + " 20:00", resourceId: 1, title: "GS1", color: "#FE6B64" },
            {
                start: days[3] + " 15:00",
                end: days[3] + " 18:00",
                resourceId: 1,
                title: "Overlapping events are positioned properly",
                color: "#779ECB",
            },

            {
                start: days[4] + " 10:00",
                end: days[4] + " 16:00",
                resourceId: 2,
                title: { html: "You have complete control over the <i><b>display</b></i> of events…" },
                color: "#779ECB",
            },
            {
                start: days[4] + " 14:00",
                end: days[4] + " 19:00",
                resourceId: 2,
                title: "…and you can drag and drop the events!",
                color: "#FE6B64",
            },
            {
                start: days[4] + " 14:00",
                end: days[6] + " 19:00",
                resourceId: 2,
                title: "LONG",
                color: "#cc6B64",
            },
            {
                start: days[5] + " 10:00",
                end: days[5] + " 16:00",
                resourceId: 2,
                title: { html: "You have complete control over the <i><b>display</b></i> of events…" },
                color: "#779ECB",
            },
            {
                start: days[5] + " 14:00",
                end: days[5] + " 19:00",
                resourceId: 2,
                title: "…and you can drag and drop the events!",
                color: "#FE6B64",
            },
            {
                start: days[5] + " 14:00",
                end: days[5] + " 19:00",
                resourceId: 2,
                title: "END",
                color: "#00cc66",
            },

            {
                start: days[6] + " 14:00",
                end: days[6] + " 19:00",
                resourceId: 2,
                title: "LAST",
                color: "red",
            },
            /*
            {
                start: days[5] + " 14:00",
                end: days[5] + " 19:00",
                resourceId: 2,
                title: "…and you can drag and drop the events!",
                color: "#cc6B64",
            },
*/
            { start: days[5] + " 18:00", end: days[6] + " 21:00", resourceId: 2, title: "GS2", color: "#B29DD9" },
            { start: days[5] + " 18:00", end: days[7] + " 21:00", resourceId: 2, title: "GS3", color: "#99ffff" },
            { start: days[7] + " 18:00", end: days[8] + " 21:00", resourceId: 2, title: "GS4", color: "#c0c0c0" },
            {
                start: days[0] + " 10:00",
                end: days[0] + " 10:00",
                resourceId: 1,
                title: "All-day 1",
                color: "#B29DD9",
                allDay: true,
            },
            {
                start: days[1] + " 10:00",
                end: days[4] + " 10:00",
                resourceId: 1,
                title: "All-day 2",
                color: "#B29DD9",
                allDay: true,
            },
            {
                start: days[3] + " 10:00",
                end: days[4] + " 10:00",
                resourceId: 1,
                title: "All-day 3",
                color: "#ff9999",
                allDay: true,
            },
            {
                start: days[4] + " 10:00",
                end: days[8] + " 10:00",
                resourceId: 1,
                title: "All-day 4",
                color: "#66ffff",
                allDay: true,
            },
        ];
    }

    function _pad(num) {
        let norm = Math.floor(Math.abs(num));
        return (norm < 10 ? "0" : "") + norm;
    }
</script>

<button onclick={toggleAllDay}>{options.allDaySlot ? "Hide" : "Show"} all-day</button>

<Calendar {plugins} {options} />

<!--
<style>
@import './global.css';
</style>
-->
