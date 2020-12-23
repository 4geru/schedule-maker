<template>
  <div class="container">
    <div>
      <h1 class="title">
        schedule-maker
      </h1>
      <div class="links">
        <label>
          日付
          <a-date-picker
            ref="dateInput"
            v-model="date"
            placeholder="開催日"
            @input="updateDate"
          />
        </label>
        <label>
          タイトル
          <a-input
            ref="titleInput"
            v-model="title"
            placeholder="google calendarのタイトル：「1200→12:00開始」「12:00-13:00→12:00開始-13:00終了」"
          >
            <a-tooltip slot="suffix" title="google calendarのタイトルです。。数字が入っている場合は、開始時間-終了時間として判定されます">
              <a-icon type="info-circle" />
            </a-tooltip>
          </a-input>
        </label>
        <label>
          説明
          <a-input
            ref="detailInput"
            v-model="detail"
            placeholder="google calendarの説明：「1200→12:00開始」「12:00-13:00→12:00開始-13:00終了」"
            @input="updateTime"
          >
            <a-tooltip slot="suffix" title="数字が入っている場合は、開始時間-終了時間として判定されます。タイトルに数字が入っている場合はタイトルの方が優先されます">
              <a-icon type="info-circle" />
            </a-tooltip>
          </a-input>
        </label>
        <label>
          所用時間
          <a-input
            ref="timeInput"
            v-model="time"
            placeholder="1.5→「開始時間から1時間30分の予定が作成されます」"
            @input="updateTime"
          >
            <a-tooltip slot="suffix" title="開始時間のみ登録されている場合に登録されます。">
              <a-icon type="info-circle" />
            </a-tooltip>
          </a-input>
        </label>
        <label>
          予定の場所
          <a-input ref="detailInput" v-model="address" placeholder="予定の場所">
            <a-tooltip slot="suffix" title="住所や場所を登録できます">
              <a-icon type="info-circle" />
            </a-tooltip>
          </a-input>
        </label>
        <label>
          URL
          <a-input ref="detailInput" v-model="url" placeholder="URL">
            <a-tooltip slot="suffix" title="URLを追加することができます">
              <a-icon type="info-circle" />
            </a-tooltip>
          </a-input>
        </label>
        <br />
        <label>
          Google calendar URL
          <a-textarea
            :value="`${calendarUrl}`"
          />
        </label>
        <a-button
          type="primary"
          icon="search"
          @click="updateTime"
        >
          Search
        </a-button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Button, Input, DatePicker } from 'ant-design-vue'
import moment from 'moment'

const DEFAULT_TIME = 1.5

export default Vue.extend({
  components: {
    Button,
    Input,
    DatePicker,
  },
  data() {
    return {
      title: '',
      date: moment().startOf('days'),
      detail: '',
      time: DEFAULT_TIME,
      startScheduledDate: moment().startOf('hours'),
      endScheduledDate: moment().startOf('hours').add(DEFAULT_TIME, 'hours'),
      address: '',
      url: '',
      calendarUrl: 'https://www.google.com/calendar/render?action=TEMPLATE&text=件名&dates=20200501T120000/20200501T140000&location=東京都千代田区霞ヶ関1-1-1&trp=true&trp=undefined&trp=true&sprop='
    }
  },
  watch: {
    startScheduledDate: function(val) {
      this.calendarUrl = `https://www.google.com/calendar/render?action=TEMPLATE&text=件名&dates=${this.startScheduledDate.format('YYYYMMDDThhmmss')}/20200501T140000&location=東京都千代田区霞ヶ関1-1-1&trp=true&trp=undefined&trp=true&sprop=`
    }
  },
  methods: {
    asyncCalendarUrl(): void {
      this.calendarUrl = `https://www.google.com/calendar/render?action=TEMPLATE&text=件名&dates=${this.startScheduledDate.format('YYYYMMDDThhmmss')}/20200501T140000&location=東京都千代田区霞ヶ関1-1-1&trp=true&trp=undefined&trp=true&sprop=`
    },
    updateDate(): void {
      this.updateDateObject(this.startScheduledDate)
      this.updateDateObject(this.endScheduledDate)
      console.log({called: 'called', startScheduledDate: this.startScheduledDate.toISOString(), endScheduledDate: this.endScheduledDate.toISOString()})
    },
    updateDateObject(date: moment): void {
      date.year(this.date.year)
      date.month(this.date.month)
      date.day(this.date.day)
    },
    updateTime(): void {
      this.convertTime()
    },
    convertTime(): number[] {
      const res = this.detail.replace(/[^0-9]/g, '')
      const splitedTime = (res.match(/.{2}/g) || []).map(e => parseInt(e))

      if (splitedTime[0]) { this.startScheduledDate.hour(splitedTime[0]) }
      if (splitedTime[1]) { this.startScheduledDate.minute(splitedTime[1]) }

      this.endScheduledDate.hour(this.startScheduledDate.hours() + (splitedTime[2] || parseInt(this.time)))
      this.endScheduledDate.minute(this.startScheduledDate.minutes() + (splitedTime[3] || (this.time % 1 * 60)))

      this.calendarUrl = `https://www.google.com/calendar/render?action=TEMPLATE&text=件名&dates=${this.startScheduledDate.format('YYYYMMDDThhmmss')}/20200501T140000&location=東京都千代田区霞ヶ関1-1-1&trp=true&trp=undefined&trp=true&sprop=`

      return splitedTime
    }
  }
})
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family:
    'Quicksand',
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}

.ant-calendar-picker {
  width: 100%;
}
</style>
