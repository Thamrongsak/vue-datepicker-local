<template>
<div class="datepicker" :class="{'datepicker-range':range,'datepicker__clearable':clearable&&text&&!disabled}">
  <input readonly :value="text" :class="[show ? 'focus' : '', inputClass]" :disabled="disabled" :placeholder="placeholder" :name="name" v-if="type!=='inline'"/>
  <a class="datepicker-close" @click.stop="cls"></a>
  <transition name="datepicker-anim">
    <div class="datepicker-popup" :class="[popupClass,{'datepicker-inline':type==='inline'}, ranges ? 'ranges': '']" tabindex="-1" v-if="show||type==='inline'">
      <template v-if="range">
        <template>
          <calendar-ranges 
            @rangeSelected="selectRange" 
            :ranges="ranges"
          ></calendar-ranges>
        </template>
        <vue-datepicker-local-calendar v-model="dates[0]" :left="true" class="left"></vue-datepicker-local-calendar>
        <vue-datepicker-local-calendar v-model="dates[1]" :right="true"></vue-datepicker-local-calendar>
      </template>
      <template v-else>
        <vue-datepicker-local-calendar v-model="dates[0]"></vue-datepicker-local-calendar>
      </template>
      <div v-if="showButtons" class="datepicker__buttons">
        <button @click.prevent.stop="cancel" class="datepicker__button-cancel">{{this.local.cancelTip}}</button>
        <button @click.prevent.stop="submit" class="datepicker__button-select">{{this.local.submitTip}}</button>
      </div>
    </div>
  </transition>
</div>
</template>

<script>
import moment from 'moment'
import VueDatepickerLocalCalendar from './VueDatepickerLocalCalendar.vue'
import CalendarRanges from './CalendarRanges.vue'
import { nextMonth, prevMonth, thisMonth, thisYear, lastWeek, lastMonth, getQuarterRange } from './util'

export default {
  name: 'VueDatepickerLocal',
  components: { VueDatepickerLocalCalendar, CalendarRanges },
  props: {
    name: [String],
    inputClass: [String],
    popupClass: [String],
    value: [Date, Array, String],
    disabled: [Boolean],
    type: {
      type: String,
      default: 'normal'
    },
    rangeSeparator: {
      type: String,
      default: '~'
    },
    clearable: {
      type: Boolean,
      default: false
    },
    placeholder: [String],
    disabledDate: {
      type: Function,
      default: () => {
        return false
      }
    },
    format: {
      type: String,
      default: 'YYYY-MM-DD'
    },
    local: {
      type: Object,
      default () {
        return {
          dow: 1, // Monday is the first day of the week
          hourTip: 'Select Hour', // tip of select hour
          minuteTip: 'Select Minute', // tip of select minute
          secondTip: 'Select Second', // tip of select second
          yearSuffix: '', // format of head
          monthsHead: 'January_February_March_April_May_June_July_August_September_October_November_December'.split('_'), // months of head
          months: 'Jan_Feb_Mar_Apr_May_Jun_Jul_Aug_Sep_Oct_Nov_Dec'.split('_'), // months of panel
          weeks: 'Su_Mo_Tu_We_Th_Fr_Sa'.split('_'), // weeks
          cancelTip: 'Cancel', // default text for cancel button
          submitTip: 'Submit' // default text for submit button
        }
      }
    },
    ranges: {
      type: Object,
      default () {
        return {
          'This month': new Date(),
          'This year': new Date(),
          'Last week': new Date(),
          'Last month': new Date(),
        }
      }
    },
    showButtons: {
      type: Boolean,
      default: false
    },
    dateRangeSelect: [Function]
  },
  data () {
    return {
      show: false,
      dates: this.vi(this.value)
    }
  },
  computed: {
    range () {
      return this.dates.length === 2
    },
    text () {
      const val = this.value
      const txt = this.dates.map(date => this.tf(date)).join(` ${this.rangeSeparator} `)
      if (Array.isArray(val)) {
        return val.length > 1 ? txt : ''
      } else {
        return val ? txt : ''
      }
    }
  },
  watch: {
    value (val) {
      this.dates = this.vi(this.value)
    }
  },
  methods: {
    get () {
      return Array.isArray(this.value) ? this.dates : this.dates[0]
    },
    cls () {
      this.$emit('clear')
      this.$emit('input', this.range ? [] : '')
    },
    vi (val) {
      if (Array.isArray(val)) {
        return val.length > 1 ? val.map(item => new Date(item)) : [new Date(), new Date()]
      } else {
        return val ? new Array(new Date(val)) : [new Date()]
      }
    },
    ok (leaveOpened) {
      const $this = this
      $this.$emit('input', $this.get())
      !leaveOpened && !$this.showButtons && setTimeout(() => {
        $this.show = $this.range
      })
    },
    tf (time, format) {
      const year = time.getFullYear()
      const month = time.getMonth()
      const day = time.getDate()
      const hours24 = time.getHours()
      const hours = hours24 % 12 === 0 ? 12 : hours24 % 12
      const minutes = time.getMinutes()
      const seconds = time.getSeconds()
      const milliseconds = time.getMilliseconds()
      const dd = t => ('0' + t).slice(-2)
      const map = {
        YYYY: year,
        MM: dd(month + 1),
        MMM: this.local.months[month],
        MMMM: this.local.monthsHead[month],
        M: month + 1,
        DD: dd(day),
        D: day,
        HH: dd(hours24),
        H: hours24,
        hh: dd(hours),
        h: hours,
        mm: dd(minutes),
        m: minutes,
        ss: dd(seconds),
        s: seconds,
        S: milliseconds
      }
      return (format || this.format).replace(/Y+|M+|D+|H+|h+|m+|s+|S+/g, str => map[str])
    },
    dc (e) {
      this.show = this.$el.contains(e.target) && !this.disabled
    },
    submit () {
      this.$emit('confirm', this.get())
      this.show = false
    },
    cancel () {
      this.$emit('cancel')
      this.show = false
    },
    setRangeType () {
        console.log('setRangeType', this.get())
    //   this.$emit('cancel')
    //   this.show = false
    },
    selectRange({rangeType, rangeData}) {
      let thisDateRange = {}
      switch( rangeType ) {
        case 'This month':
          thisDateRange = thisMonth()
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        case 'This year':
          thisDateRange = thisYear()
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        case 'Last week':
          thisDateRange = lastWeek()
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        case 'Last month':
          thisDateRange = lastMonth()
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        case 'Quarter 1':
          thisDateRange = getQuarterRange(1, this.dates)
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        case 'Quarter 2':
          thisDateRange = getQuarterRange(2, this.dates)
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        case 'Quarter 3':
          thisDateRange = getQuarterRange(3, this.dates)
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        case 'Quarter 4':
          thisDateRange = getQuarterRange(4, this.dates)
          this.dates = [thisDateRange.start, thisDateRange.end]
          if(!this.showButtons){
            this.show = false
          }
          break;
        default:
          this.dates = rangeData
          if(!this.showButtons){
            this.show = false
          }
          break;
      }
    }
  },
  mounted () {
    document.addEventListener('click', this.dc, true)
  },
  beforeDestroy () {
    document.removeEventListener('click', this.dc, true)
  }
}
</script>

<style>
.datepicker {
  display: inline-block;
  position: relative;
}

.datepicker:before {
  content: '';
  display: block;
  position: absolute;
  width: 34px;
  height: 100%;
  top: 0;
  right: 0;
  background: url('data:image/svg+xml;base64,PHN2ZyBjbGFzcz0iaWNvbiIgdmlld0JveD0iMCAwIDEwMjQgMTAyNCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB3aWR0aD0iMjAiIGhlaWdodD0iMjAiPjxwYXRoIGQ9Ik01NjQgMTgwLjJINDQ4Yy04LjMgMC0xNS02LjctMTUtMTVzNi43LTE1IDE1LTE1aDExNmM4LjIgMCAxNSA2LjcgMTUgMTVzLTYuOCAxNS0xNSAxNXoiIGZpbGw9IiM5ODk4OTgiLz48cGF0aCBkPSJNOTQ1IDk1Mi4ySDgxLjJjLTguMiAwLTE1LTYuNy0xNS0xNVYxNjIuOGMwLTguMyA2LjgtMTUgMTUtMTVIMjk0YzguMiAwIDE1IDYuNyAxNSAxNXMtNi44IDE1LTE1IDE1SDk2LjJ2NzQ0LjRIOTMwVjE3Ny44SDcxMy42Yy04LjMgMC0xNS02LjctMTUtMTVzNi43LTE1IDE1LTE1SDk0NWM4LjIgMCAxNSA2LjcgMTUgMTV2Nzc0LjRjMCA4LjMtNi44IDE1LTE1IDE1eiIgZmlsbD0iIzk4OTg5OCIvPjxwYXRoIGQ9Ik0zMzMuMyA1NTFIMjE2Yy04LjIgMC0xNS02LjgtMTUtMTVzNi44LTE1IDE1LTE1aDExNy4zYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1em0yMzAuMyAwSDQ0Ni4zYy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4zYzguMiAwIDE1IDYuNiAxNSAxNXMtNi44IDE1LTE1IDE1em0yMzAuMiAwSDY3Ni42Yy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4yYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1ek0zMzMuMyA3NDBIMjE2Yy04LjIgMC0xNS02LjgtMTUtMTVzNi44LTE1IDE1LTE1aDExNy4zYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1em0yMzAuMyAwSDQ0Ni4zYy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4zYzguMiAwIDE1IDYuNiAxNSAxNXMtNi44IDE1LTE1IDE1em0yMzAuMiAwSDY3Ni42Yy04LjMgMC0xNS02LjgtMTUtMTVzNi43LTE1IDE1LTE1aDExNy4yYzguMyAwIDE1IDYuNiAxNSAxNXMtNi43IDE1LTE1IDE1ek0zNzAuOCAyNTguNmMtOC4zIDAtMTUtNi43LTE1LTE1Vjg2LjhjMC04LjIgNi43LTE1IDE1LTE1czE1IDYuOCAxNSAxNXYxNTYuOGMwIDguMy02LjcgMTUtMTUgMTV6bTI3MC4yIDBjLTguMyAwLTE1LTYuNy0xNS0xNVY4Ni44YzAtOC4yIDYuNy0xNSAxNS0xNXMxNSA2LjggMTUgMTV2MTU2LjhjMCA4LjMtNi43IDE1LTE1IDE1ek05NDUgMzcyLjJIODEuMmMtOC4yIDAtMTUtNi43LTE1LTE1czYuOC0xNSAxNS0xNUg5NDVjOC4yIDAgMTUgNi43IDE1IDE1cy02LjggMTUtMTUgMTV6IiBmaWxsPSIjOTg5ODk4Ii8+PC9zdmc+') no-repeat 50% 50%;
}

.datepicker-close {
  display: none;
  position: absolute;
  width: 34px;
  height: 100%;
  top: 0;
  right: 0;
  cursor: pointer;
}

.datepicker-close:before {
  display: block;
  content: '';
  position: absolute;
  width: 16px;
  height: 16px;
  left: 50%;
  top: 50%;
  margin-left: -8px;
  margin-top: -8px;
  text-align: center;
  background: #ccc;
  color: #fff;
  border-radius: 50%;
  background:#ccc url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA3IDciIHdpZHRoPSI3IiBoZWlnaHQ9IjciPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik01LjU4LDVsMi44LTIuODFBLjQxLjQxLDAsMSwwLDcuOCwxLjZMNSw0LjQxLDIuMiwxLjZhLjQxLjQxLDAsMCwwLS41OC41OGgwTDQuNDIsNSwxLjYyLDcuOGEuNDEuNDEsMCwwLDAsLjU4LjU4TDUsNS41OCw3LjgsOC4zOWEuNDEuNDEsMCwwLDAsLjU4LS41OGgwWiIgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoLTEuNSAtMS40OCkiIHN0eWxlPSJmaWxsOiNmZmYiLz48L3N2Zz4NCg==') no-repeat 50% 50%;
}

.datepicker__clearable:hover:before {
  display: none;
}
.datepicker__clearable:hover .datepicker-close{
  display: block;
}

.datepicker-close:hover:before{
  background-color: #afafaf;
}

.datepicker>input {
  color: #666;
  transition: all 200ms ease;
  border: 1px solid #e5e5e5;
  height: 34px;
  box-sizing: border-box;
  outline: none;
  padding: 0 34px 0 12px;
  font-size: 14px;
  width: 100%;
  user-select: none;
  -ms-user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
}

.datepicker>input.focus {
  border-color: #3bb4f2;
  -webkit-box-shadow: 0 0 5px rgba(59, 180, 242, .3);
  box-shadow: 0 0 5px rgba(59, 180, 242, .3);
}

.datepicker>input:disabled {
  cursor: not-allowed;
  background-color: #ebebe4;
  border-color: #e5e5e5;
  -webkit-box-shadow: none;
  box-shadow: none;
}

.datepicker-popup {
  position: absolute;
  transition: all 200ms ease;
  opacity: 1;
  transform: scaleY(1);
  transform-origin: center top;
  font-size: 12px;
  background: #fff;
  border: 1px solid #d9d9d9;
  box-shadow: 0 1px 6px rgba(99, 99, 99, 0.2);
  margin-top: 2px;
  outline: 0;
  padding: 5px;
  overflow: hidden;
  z-index: 999
}

.datepicker-inline{
  position: relative;
  margin-top: 0;
}

.datepicker-range {
  min-width: 325px
}

.datepicker-range .datepicker-popup{
  width: 403px;
}
.datepicker-range .datepicker-popup.ranges{
  width: 512px;
}

.datepicker-bottom {
  float: left;
  width: 100%;
  text-align: right;
}

.datepicker-btn {
  padding: 5px 10px;
  background: #1284e7;
  color: #fff;
  border-radius: 2px;
  display: inline-block;
  cursor: pointer;
}
.datepicker-anim-enter-active {
    transform-origin: 0 0;
    animation: datepicker-anim-in .2s cubic-bezier(.23, 1, .32, 1)
}

.datepicker-anim-leave-active {
    transform-origin: 0 0;
    animation: datepicker-anim-out .2s cubic-bezier(.755, .05, .855, .06)
}

.datepicker__buttons {
  display: block;
  text-align: right;
}

.datepicker__buttons button {
  display: inline-block;
  font-size: 13px;
  border: none;
  cursor: pointer;
  margin: 10px 0 0 5px;
  padding: 5px 15px;
  color: #ffffff;
}

.datepicker__buttons .datepicker__button-select {
  background: #1284e7;
}

.datepicker__buttons .datepicker__button-cancel {
  background: #666;
}

@keyframes datepicker-anim-in {
    0% {
        opacity: 0;
        transform: scaleY(.8)
    }
    to {
        opacity: 1;
        transform: scaleY(1)
    }
}

@keyframes datepicker-anim-out {
    0% {
        opacity: 1;
        transform: scaleY(1)
    }
    to {
        opacity: 0;
        transform: scaleY(.8)
    }
}
</style>
