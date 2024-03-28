<template>
    <div class="content">
        <h2>カレンダー{{ displayDate }}</h2>
        <div class="button-area">
            <button class="button" @click="prevMonth">前の月</button>
            <button class="button" @click="nextMonth">次の月</button>
        </div>
        <div class="calendar">
            <div class="calendar-weekly">
                <div class="calendar-youbi" v-for="n in 7" :key="n">
                    {{ youbi(n - 1) }}
                </div>
            </div>
            <div v-for="(week, index) in calendars" :key="index" class="calendar-weekly">
                <div v-for="(day, index) in week" :key="index"
                    class="calendar-daily"
                    :class="{ outside: currentMonth !== day.month }"
                    @drop="dragEnd($event, day.date)"
                    @dragover.prevent
                >
                    <div class="calendar-day">
                        {{ day.day }}
                    </div>
                    <div v-for="dayEvent in day.dayEvents" :key="dayEvent.id">
                        <div 
                            v-if="dayEvent.width"
                            class="calendar-event"
                            :style="`width:${dayEvent.width}%;background-color:${dayEvent.color}`"
                            draggable="true"
                            @dragstart="dragStart($event, dayEvent.id)"
                        >{{ dayEvent.name }}</div>
                        <div v-else style="height:26px"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import moment from 'moment';

export default {
    name: 'Calendar',
    data() {
        return {
            currentDate: moment(),
            events: [
                { id: 1, name: "ミーティング", start: "2024-03-01", end: "2024-03-01", color: "blue" },
                { id: 2, name: "イベント", start: "2024-03-02", end: "2024-03-03", color: "limegreen" },
                { id: 3, name: "会議", start: "2024-03-06", end: "2024-03-06", color: "deepskyblue" },
                { id: 4, name: "有給", start: "2024-03-08", end: "2024-03-08", color: "dimgray" },
                { id: 5, name: "海外旅行", start: "2024-03-08", end: "2024-03-11", color: "navy" },
                { id: 6, name: "誕生日", start: "2024-03-16", end: "2024-03-16", color: "orange" },
                { id: 7, name: "イベント", start: "2024-03-12", end: "2024-03-15", color: "limegreen" },
                { id: 8, name: "出張", start: "2024-03-12", end: "2024-03-13", color: "teal" },
                { id: 9, name: "客先訪問", start: "2024-03-14", end: "2024-03-14", color: "red" },
                { id: 10, name: "パーティ", start: "2024-03-15", end: "2024-03-15", color: "royalblue" },
                { id: 12, name: "ミーティング", start: "2024-03-18", end: "2024-03-19", color: "blue" },
                { id: 13, name: "イベント", start: "2024-03-21", end: "2024-03-21", color: "limegreen" },
                { id: 14, name: "有給", start: "2024-03-20", end: "2024-03-20", color: "dimgray" },
                { id: 15, name: "イベント", start: "2024-03-25", end: "2024-03-28", color: "limegreen" },
                { id: 16, name: "会議", start: "2024-03-21", end: "2024-03-21", color: "deepskyblue" },
                { id: 17, name: "旅行", start: "2024-03-23", end: "2024-03-24", color: "navy" },
                { id: 18, name: "ミーティング", start: "2024-03-28", end: "2024-03-28", color: "blue" },
                { id: 19, name: "会議", start: "2024-03-12", end: "2024-03-12", color: "deepskyblue" },
                { id: 20, name: "誕生日", start: "2024-03-30", end: "2024-03-30", color: "orange" },
            ]
        }
    },
    methods: {
        getStartDate() {
            let date = moment(this.currentDate);
            date.startOf("month");
            const youbiNm = date.day();
            return date.subtract(youbiNm, "days");
        },
        getEndDate() {
            let date = moment(this.currentDate);
            date.endOf("month");
            const youbiNm = date.day();
            return date.add(6 - youbiNm, "days");
        },
        getCalendar() {
            let startDate = this.getStartDate();
            const endDate = this.getEndDate();
            const weekNumber = Math.ceil(endDate.diff(startDate, "days") / 7);

            let calendars = [];
            let calendarDate = this.getStartDate();
            for (let week = 0; week < weekNumber; week++) {
                let weekRow = [];
                for (let day = 0; day < 7; day++) {
                    let dayEvents = this.getDayEvents(calendarDate, day);
                    weekRow.push({
                        day: calendarDate.get("date"),
                        month: calendarDate.format("YYYY-MM"),
                        date: calendarDate.format("YYYY-MM-DD"),
                        dayEvents,
                    });
                    calendarDate.add(1, "days");
                }
                calendars.push(weekRow);
            }
            return calendars;
        },
        nextMonth() {
            this.currentDate = moment(this.currentDate).add(1, "month");
        },
        prevMonth() {
            this.currentDate = moment(this.currentDate).subtract(1, "month");
        },
        youbi(dayIndex) {
            const week = ["日", "月", "火", "水", "木", "金", "土"];
            return week[dayIndex];
        },
        getDayEvents(date, day) {
            let stackIndex = 0;
            let dayEvents = [];
            let startedEvents = [];
            this.sortedEvents.forEach(event => {
                let startDate = moment(event.start).format("YYYY-MM-DD");
                let endDate = moment(event.end).format("YYYY-MM-DD");
                let _date = date.format("YYYY-MM-DD");
                if(startDate <= _date && endDate >= _date) {
                    if(startDate === _date) {
                        [stackIndex, dayEvents] = this.getStackEvents(event, day, stackIndex, dayEvents, startedEvents, event.start);
                    } else if(day === 0) {
                        [stackIndex, dayEvents] = this.getStackEvents(event, day, stackIndex, dayEvents, startedEvents, _date);
                    } else {
                        startedEvents.push(event);
                    }
                }
            });
            return dayEvents;
        },
        getEventWidth(start, end, day){
            let betweenDays = moment(end).diff(moment(start), "days");
            if(betweenDays > 6 - day) {
                return (6 - day) * 100 + 95;
            } else {
                return betweenDays * 100 + 95;
            }
        },
        getStackEvents(event, day, stackIndex, dayEvents, startedEvents, start){
            [stackIndex, dayEvents] = this.getStartedEvents(stackIndex, startedEvents, dayEvents)
            let width = this.getEventWidth(start, event.end, day)
            Object.assign(event,{
                stackIndex
            })
            dayEvents.push({...event, width})
            stackIndex++;
            return [stackIndex,dayEvents];
        },
        getStartedEvents(stackIndex, startedEvents, dayEvents){
            let startedEvent;
            do{
                startedEvent = startedEvents.find(event => event.stackIndex === stackIndex)
                if(startedEvent) {
                dayEvents.push(startedEvent)　//ダミー領域として利用するため
                stackIndex++;
                }
            }while(typeof startedEvent !== 'undefined')
                return [stackIndex, dayEvents]
        },
        dragStart(event, eventId) {
            event.dataTransfer.effectAllowed = "move";
            event.dataTransfer.dropEffect = "move";
            event.dataTransfer.setData("eventId", eventId);
        },
        dragEnd(event, date){
            console.log(event);
            console.log(date);
            let eventId = event.dataTransfer.getData("eventId");
            let dragEvent = this.events.find(event => event.id == eventId);
            let betweenDays = moment(dragEvent.end).diff(moment(dragEvent.start), "days");
            dragEvent.start = date;
            dragEvent.end = moment(dragEvent.start).add(betweenDays, "days").format("YYYY-MM-DD");
        }
    },
    computed: {
        calendars() {
            return this.getCalendar();
        },
        displayDate() {
            return this.currentDate.format("YYYY年MM月");
        },
        currentMonth() {
            return this.currentDate.format("YYYY-MM");
        },
        sortedEvents() {
            return this.events.slice().sort(function(a, b) {
                let startDate = moment(a.start).format("YYYY-MM-DD");
                let startDate2 = moment(b.start).format("YYYY-MM-DD");
                if(startDate < startDate2) return -1;
                if(startDate > startDate2) return 1;
                return 0;
            })
        },
    },
    mounted() {
        console.log(this.getCalendar());
    },

}
</script>

<style>
.content {
    margin: 2em auto;
    width: 900px;
}

.button-area {
    margin: 0.5em 0;
}

.button {
    padding: 4px 8px;
    margin-right: 8px;
}

.calendar {
    max-width: 900px;
    border-top: 1px solid #E0E0E0;
    font-size: 0.8em;
}

.calendar-weekly {
    display: flex;
    border-left: 1px solid #E0E0E0;
    /* background-color: black; */
}

.calendar-daily {
    flex: 1;
    min-height: 125px;
    border-right: 1px solid #E0E0E0;
    border-bottom: 1px solid #E0E0E0;
    margin-right: -1px;
}

.calendar-day {
    text-align: center;
}

.calendar-youbi {
    flex: 1;
    border-right: 1px solid #E0E0E0;
    margin-right: -1px;
    text-align: center;
}

.outside {
    background-color: #f7f7f7;
}

.calendar-event {
    color: white;
    margin-bottom: 1px;
    height: 25px;
    line-height: 25px;
    position: relative;
    z-index: 1;
    border-radius: 4px;
    padding-left: 4px;
}
</style>