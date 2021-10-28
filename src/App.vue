<template>
  <div id="wrapCalendar" class="wrap-calendar">
    <header>
      <h1>캘린더</h1>
      <div class="area-arrow">
        <button class="prev">이전</button>
        <button class="next">다음</button>
      </div>
      <h2>
        {{ year }}년 {{ monthForView }}월
      </h2>
    </header>
    <div id="areaCalendar" class="area-calendar">
      <table class="calendar">
        <caption>캘린더</caption>
        <thead>
          <tr>
            <th>일</th>
            <th>월</th>
            <th>화</th>
            <th>수</th>
            <th>목</th>
            <th>금</th>
            <th>토</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(schedules, index1) in schedulesByWeek"
            :key="`tr-${index1}`"
          >
            <td
              v-for="(schedule, index2) in schedules"
              :key="`td-${index1}-${index2}`"
              :class="{'today': isToday(schedule.date)}"
            >
              <button 
                v-if="schedule.date"
                class="add-schedule"
                :data-date="schedule.date"
              >
                <span class="date">{{ schedule.date }}</span>
                <span class="button-text">+ SCHEDULE</span>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
// import axios from 'axios'
import ScheduleData from './data/schedules.json'

export default {
  name: 'Vue Calendar',
  components: {
  },
  data() {
    return {
      year: new Date().getFullYear(),
      month: new Date().getMonth(),
      rowTotal: 6,
      weekDays: 7,
      schedules: [],
    }
  },
  computed: {
    monthForView() {
      return this.month + 1
    },
    firstDay() {
      return new Date(this.year, this.month, 1).getDay()
    },
    lastDate() {
      const is31Months = [1,3,5,7,8,10,12].includes(this.month)
      const is30Months = [4,6,9,11].includes(this.month)
      const 윤년 = this.year % 4 === 0

      return is31Months ? 31
          : is30Months ? 30
          : 2 === this.month && (윤년 ? 29 : 28)
    },
    thisMonthSchedules() {
      // 여기 다시 정리
      let thisMonthSchedules = {}
      for(const schedule of this.schedules) {
        const isThisMonthSchedule = this.year === schedule.year 
                                    && this.month+1 === schedule.month
        if(isThisMonthSchedule){
          const scheduleFirstDate = schedule.date[0]
          if(!Array.isArray(thisMonthSchedules[scheduleFirstDate])) thisMonthSchedules[scheduleFirstDate] = []
          thisMonthSchedules[scheduleFirstDate].push(schedule)
        }
      }
      console.log(thisMonthSchedules)
      return thisMonthSchedules
    },

    schedulesByWeek() {
      const schedulesByWeek = []
      const tdTotal = this.rowTotal * this.weekDays

      for(let i = 0; i <= tdTotal - 1; i++) {
        const rowNumber = parseInt(i / this.weekDays)
        const date = i - this.firstDay
        const isOutDate = date < 0 || this.lastDate < date     
        const schedule = {
          date: isOutDate ? null : date + 1,
          day: i%7,
          plans: []
        }
        // 저장된 스케줄과 날짜 동일한 경우 관련 데이터 추가
        const thisDateSchedules = this.thisMonthSchedules[date]    
        if(thisDateSchedules) {
          for(const thisDateSchedule of thisDateSchedules) {
            schedule.description = thisDateSchedule.description
            schedule.title = thisDateSchedule.title
            schedule.dayLong = thisDateSchedule.date.length
          }
        }
        // 스케줄 리스트 업
        !schedulesByWeek[rowNumber] && schedulesByWeek.push([])
        schedulesByWeek[rowNumber].push(schedule)
      }

      return schedulesByWeek 
    },
  },
  watch() {

  },
  created() {
    this.schedules = ScheduleData
  },
  mounted() {
    console.log(this.thisMonthSchedules)
    console.log(this.schedulesByWeek)
  },
  methods: {
    isToday(date) {      
      const isThisYear = new Date().getFullYear() === this.year
      const isThisMonth = new Date().getMonth() === this.month
      const isToday = new Date().getDate() === date
      return isThisYear && isThisMonth && isToday && true
    },
    dayOfWeek(day) {
      const dayOfWeek = ['일요일', '월요일', '화요일', '수요일', '목요일', '금요일', '토요일']
      return dayOfWeek[day]
    }
  }
}
</script>

<style lang="scss">
@import './assets/style/common';
@import './assets/style/calendar';
</style>
