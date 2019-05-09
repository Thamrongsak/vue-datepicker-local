
<template>
    <div class="ranges">
        <ul v-if="ranges">
            <li v-for="(value, key) in ranges" :key="key" @click="selectedRange(key, value)">{{key}}</li>
        </ul>
    </div>
</template>

<script>
    export default {
        name: 'CalendarRanges',
        props: ['ranges'],
        methods: {
          selectedRange(rangeType, rangeData) {
            this.$emit('rangeSelected', { rangeType, rangeData })
          }
        }
    }
</script>

<style lang="css">
.ranges {
    font-size: 11px;
    float: left;
    margin: 0px 4px 0 0;
    text-align: left;
    width: 100px;
    display: inline-block;
}
.ranges ul {
    list-style: none;
    margin: 0 auto;
    padding: 0;
    width: 100%;
}
.ranges li {
    font-size: 13px;
    padding: 6px 12px;
    margin-bottom: 3px;
    cursor: pointer;
}
.ranges li:hover {
    background-color: #eee;
}
.ranges li.active {
    background-color: #eee;
}
</style>