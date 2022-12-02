<template>
  <div>
    <div v-if="period.unit === 'month'" class="d-flex align-items-center mt-3">
      <div class="font-weight-bold mr-3">대상 연월</div>
      <div class="date-picker ml-4">
        <vue-monthly-picker
          v-model="start_date"
          selected-background-color="black"
          date-format="YYYY-MM"
          :clear-option="false"
        >
        </vue-monthly-picker>
      </div>
    </div>
    <div class="d-flex align-items-center mt-3">
      <div class="w-120px font-weight-bold">
        일괄 등록
      </div>
      <div class="px-3 py-3 pb-1 bg-grey4 border rounded w-100">
        <div class="d-flex align-items-top">
          <label class="mr-3 mt-1">대상</label>
          <div class="btn-group">
            <label
              v-for="(data, key) in controlData.day"
              :key="key"
              class="btn btn-toggle-day"
              :class="controlData.day[key] ? 'active' : ''"
            >
              <input
                v-model="controlData.day[key]"
                type="checkbox"
                class="custom-control-input"
              />
              {{ dayname(key) }}
            </label>
          </div>
        </div>
        <div class="d-flex align-items-center mt-2">
          <label class="mr-3">시간</label>
          <select
            class="form-control w-180px mr-3"
          >
            <option value="work">근로시간</option>
          </select>
          <div class="wrap-timepicker">
            <template>
              <VueCtkDateTimePicker
                id="start_time"
                v-model="controlData.start_time"
                :no-label="true"
                label=""
                format="HH:mm"
                formatted="HH:mm"
                :minute-interval="10"
                input-size="sm"
                :only-time="true"
                class="only-time"
              >
              </VueCtkDateTimePicker>
              <span class="mx-1">~</span>
              <VueCtkDateTimePicker
                id="end_time"
                v-model="controlData.end_time"
                :no-label="true"
                label=""
                format="HH:mm"
                formatted="HH:mm"
                :minute-interval="10"
                input-size="sm"
                :only-time="true"
                class="only-time"
              >
              </VueCtkDateTimePicker>
            </template>
          </div>
          <div class="ml-auto">
            <button
              type="button"
              class="btn btn-sm btn-outline-success"
              @click="controlApply()"
            >
              스케줄 반영
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="mt-3">
      <p class="font-weight-bold mb-2">
        스케줄
      </p>
      <table class="table text-center small">
        <caption>&middot; 해당 요일이 공휴일이라면 '공휴일여부'에 표기해주세요.</caption>
        <thead class="font-secondary">
          <tr>
            <th scope="col" class="w-100px">
              <span v-if="period.unit === 'month'">날짜</span>
              <span v-else>단위기간</span>
            </th>
            <th scope="col" class="w-100px">
              요일
            </th>
            <th scope="col" class="w-270px">
              출근
            </th>
            <th scope="col" class="w-270px">
              퇴근
            </th>
            <th scope="col" class="w-160px">
              공휴일여부
            </th>
            <th scope="col" class="w-100px">
              근로시간
            </th>
            <th scope="col" class="w-100px">
              휴게시간
            </th>
          </tr>
        </thead>

        <tbody v-if="period.unit === 'week'">
          <tr v-for="(week, index) in selective_working_hour_schedule" :key="index">
            <td class="w-90px"> {{ index+1 }} 주</td>
            <td colspan="6" style="padding:none">
              <table class="week-table">
                <tr v-for="(schedule, key) in week" :key="key">
                  <td scope="row" class="w-100px">{{ dayname(key) }}요일</td>
                  <td class="w-270px">
                    <div class="d-flex align-items-center justify-content-center">
                      <span><VueCtkDateTimePicker
                          :id="index + key + '-start_time'"
                          v-model="schedule.start_time"
                          :disabled="schedule.holiday === true"
                          :no-label="true"
                          label=""
                          format="HH:mm"
                          formatted="HH:mm"
                          :minute-interval="10"
                          input-size="sm"
                          :only-time="true"
                          class="only-time"
                      ></VueCtkDateTimePicker></span>
                    </div>
                  </td>
                  <td class="w-270px">
                    <div class="d-flex align-items-center justify-content-center">
                      <span><VueCtkDateTimePicker
                          :id="index + key + '-end_time'"
                          v-model="schedule.end_time"
                          :disabled="schedule.holiday === true"
                          :no-label="true"
                          label=""
                          format="HH:mm"
                          formatted="HH:mm"
                          :minute-interval="10"
                          input-size="sm"
                          :only-time="true"
                          class="only-time"
                      ></VueCtkDateTimePicker></span>
                    </div>
                  </td>
                  <td class="w-160px">
                    <div class="align-items-center form-check">
                      <input v-model="schedule.holiday" class="form-check-input" type="checkbox" value="" :id="index + key + 'holiday'">
                      <label class="form-check-label" :for="index + key + 'holiday'">공휴일</label>
                    </div>
                  </td>
                  <td class="w-100px">{{ minuteDisplay(dailyCalc(index, key)) }}</td>
                  <td class="w-100px">{{ minuteDisplay(schedule.break_min) }}</td>
                </tr>
              </table>
            </td>
          </tr>
          <tr class="alert-primary font-weight-bold border-bottom">
            <td colspan="7">
              {{ period.amount }}주
            </td>
          </tr>
        </tbody>
        <!-- 1개월일때 -->
        <tbody v-else-if="start_date !== null">
          <tr v-for="(schedule, index) in selective_working_hour_schedule" :key="index">
            <td scope="row">{{ schedule.date }}</td>
            <td>{{ displayDay(schedule.date) }}요일</td>
            <td>
              <div class="d-flex align-items-center justify-content-center">
                <span><VueCtkDateTimePicker
                    :id="index + '-start_time'"
                    v-model="schedule.start_time"
                    :disabled="schedule.holiday === true"
                    :no-label="true"
                    label=""
                    format="HH:mm"
                    formatted="HH:mm"
                    :minute-interval="10"
                    input-size="sm"
                    :only-time="true"
                    class="only-time"
                ></VueCtkDateTimePicker></span>
              </div>
            </td>
            <td>
              <div class="d-flex align-items-center justify-content-center">
                <span><VueCtkDateTimePicker
                    :id="index + '-end_time'"
                    v-model="schedule.end_time"
                    :disabled="schedule.holiday === true"
                    :no-label="true"
                    label=""
                    format="HH:mm"
                    formatted="HH:mm"
                    :minute-interval="10"
                    input-size="sm"
                    :only-time="true"
                    class="only-time"
                ></VueCtkDateTimePicker></span>
              </div>
            </td>
            <td>
              <div class="align-items-center form-check">
                <input v-model="schedule.holiday" class="form-check-input" type="checkbox" value="" :id="index + 'holiday'">
                <label class="form-check-label" :for="index + 'holiday'">공휴일</label>
              </div>
            </td>
            <td>{{ minuteDisplay(dailyCalc(index)) }}</td>
            <td>{{ minuteDisplay(schedule.break_min) }}</td>
          </tr>
          <tr class="alert-primary font-weight-bold border-bottom">
            <td colspan="7">1개월</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="d-flex align-items-center">
      <div class="font-weight-bold">총 근로시간 : </div>
      <div class="font-weight-bold ml-2">{{ minuteDisplay(totalWorktime()) }}</div>
    </div>
    <div class="d-flex align-items-center mt-2" v-if="over_working_min > 0">
      <div>( 소정 근로시간 : {{minuteDisplay(fixed_working_min)}}</div>
      <div class="ml-3 mr-3">/</div>
      <div>추가 근로시간 : {{minuteDisplay(over_working_min)}} )</div>
    </div>
    <div class="d-flex align-items-center row mb-5">
      <div class="font-weight-bold col-2">
        근로간주 및 대체
      </div>
      <div class="px-3 py-2 bg-grey4 border rounded small ml-3 col mt-4">
        <p>&middot; 출장 : 당일 소정근로시간만큼 근로 간주 (실 근로 처리)</p>
        <p>
          &middot; 휴가 : 당일 소정근로시간을 충족(대체)한 것으로 보되, 실
          근로시간으로는 처리하지 않음 <br /><span class="font-secondary ml-2"
            >(연차=당일 소정근로시간만큼, 반차/반반차=당일 소정근로시간의 ½, ¼
            소정근로 충족)</span
          >
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import VueCtkDateTimePicker from 'vue-ctk-date-time-picker';
import 'vue-ctk-date-time-picker/dist/vue-ctk-date-time-picker.css';

Vue.component('VueCtkDateTimePicker', VueCtkDateTimePicker);

export default {
  props: {
    period: {
      type: Object,
      default: function() {
        return {}
      }
    },
  },
  data() {
    return {
      controlData: {
        type: 'day',
        day: {
          mon: false,
          tue: false,
          wed: false,
          thu: false,
          fri: false,
          sat: false,
          sun: false
        }
      },
      daytitle: {
        mon: '월',
        tue: '화',
        wed: '수',
        thu: '목',
        fri: '금',
        sat: '토',
        sun: '일'
      },
      selective_working_hour_schedule: [],
      total_time: 0,
      start_date: null,
      fixed_working_min: 0,
      over_working_min: 0
    }
  },
  watch: {
    'period.amount': function() {
      this.computed_selective_working_hour_schedule()
    },
    'period.unit': function() {
      this.computed_selective_working_hour_schedule()
    },
    start_date: function() {
      this.computed_selective_working_hour_schedule()
    },
  },
  mounted() {
    this.selective_working_hour_schedule = this.computed_selective_working_hour_schedule()
  },
  methods: {
    dayname(val) {
      return this.daytitle[val]
    },
    displayDay: function(val) {
      return Vue.moment(val).format('ddd')
    },
    dayKey(val) {
      const title = Vue.moment(val).format('ddd')

      return window._.findKey(
        this.daytitle,
        window._.partial(window._.isEqual, title)
      )
    },
    countWeekday() {
      let weekday = 0

      if(typeof this.selective_working_hour_schedule !== 'undefined') {
        window._.forEach(this.selective_working_hour_schedule, val => {
          let day = Vue.moment(val.date).format('ddd')
          if (day !== this.daytitle.sat && day !== this.daytitle.sun) {
            weekday++
          }
        })
      }

      return weekday
    },
    totalWorktime() {
      let time = 0
      let fixed_min = 0

      if(typeof this.selective_working_hour_schedule !== 'undefined') {
        if(this.period.unit === 'week') {
          for (var i = 0; i < this.period.amount; i++) {
            window._.forEach(this.selective_working_hour_schedule[i], val => {
              if (val.work_min !== null) {
                time += val.work_min
              }
            })
          }

          fixed_min = 2400 * this.period.amount
        } else {
          window._.forEach(this.selective_working_hour_schedule, val => {
            if (val.work_min !== null) {
              time += val.work_min
            }
          })

          let weekday = this.countWeekday()
          fixed_min = 480 * weekday
        }
      }

      if (fixed_min < time) {
        this.fixed_working_min = fixed_min
        this.over_working_min = time - fixed_min
      } else {
        this.fixed_working_min = 0
        this.over_working_min = 0
      }

      return this.total_time = time
    },
    dailyCalc(index, key = null) {
      let total_time = 0
      let work_time = 0
      let break_time = 0
      let schedule

      if (this.period.unit === 'month') {
        schedule = this.selective_working_hour_schedule[index]
      } else {
        schedule = this.selective_working_hour_schedule[index][key]
      }

      if (schedule.start_time !== null && schedule.end_time !== null && schedule.holiday === false) {
        let start_time = Vue.moment(schedule.start_time, 'HH:mm')
        let end_time = Vue.moment(schedule.end_time, 'HH:mm')

        if (start_time > end_time) {
          const start_target = Vue.moment('24:00', 'HH:mm')
          let start_cal = start_target.diff(start_time, 'minutes')

          const end_target = Vue.moment('00:00', 'HH:mm')
          let end_cal = end_time.diff(end_target, 'minutes')

          total_time = start_cal + end_cal
        } else {
          total_time = end_time.diff(start_time, 'minutes')
        }

        work_time = total_time
        while (total_time >= 240) {
          total_time -= 270

          work_time = work_time - 30
          break_time += 30
        }
      } else if (schedule.holiday === true) {
        work_time = 480
      }

      schedule.work_min = work_time
      schedule.break_min = break_time

      if (this.period.unit === 'month') {
        this.selective_working_hour_schedule[index] = schedule
      } else {
        this.selective_working_hour_schedule[index][key] = schedule
      }

      return work_time
    },
    minuteDisplay(minute) {
      if (!isNaN(minute)) {
        let text = '-'

        if (minute !== 0) {
          text = `${minute / 60 >= 1 ? Math.floor(minute / 60) + '시간' : ''}${
            minute % 60 !== 0 ? Math.floor(minute % 60) + '분' : ''
          }`
        }

        return text
      } else {
        return '0시간'
      }
    },
    controlApply() {
      const data = this.controlData

      window._.forEach(data.day, (val, key) => {
        // 체크여부 확인
        if (val === false) {
          return
        }

        if (data.start_time !== undefined) {
          if (this.period.unit === 'week') {
            window._.forEach(this.selective_working_hour_schedule, (val) => {
              val[key].start_time = data.start_time
            })
          } else {
            window._.forEach(this.selective_working_hour_schedule, formVal => {
              if (this.dayKey(formVal.date) === key) {
                formVal.start_time = data.start_time
              }
            })
          }
        }

        if (data.end_time !== undefined) {
          if (this.period.unit === 'week') {
            window._.forEach(this.selective_working_hour_schedule, (val) => {
              val[key].end_time = data.end_time
            })
          } else {
            window._.forEach(this.selective_working_hour_schedule, formVal => {
              if (this.dayKey(formVal.date) === key) {
                formVal.end_time = data.end_time
              }
            })
          }
        }
      })
    },
    computed_selective_working_hour_schedule() {
      if (this.period !== null && this.period.unit === 'week') {
        this.selective_working_hour_schedule = []

        for (var i = 0; i < this.period.amount; i++) {
          this.selective_working_hour_schedule.push({
            mon: {
              start_time: null,
              end_time: null,
              holiday: false,
              work_min: 0,
              break_min: 0
            },
            tue: {
              start_time: null,
              end_time: null,
              holiday: false,
              work_min: 0,
              break_min: 0
            },
            wed: {
              start_time: null,
              end_time: null,
              holiday: false,
              work_min: 0,
              break_min: 0
            },
            thu: {
              start_time: null,
              end_time: null,
              holiday: false,
              work_min: 0,
              break_min: 0
            },
            fri: {
              start_time: null,
              end_time: null,
              holiday: false,
              work_min: 0,
              break_min: 0
            },
            sat: {
              start_time: null,
              end_time: null,
              holiday: false,
              work_min: 0,
              break_min: 0
            },
            sun: {
              start_time: null,
              end_time: null,
              holiday: false,
              work_min: 0,
              break_min: 0
            },
          });
        }
      } else if (this.period !== null && this.start_date !== null) {
        this.selective_working_hour_schedule = []
        var firstDate = Vue.moment(this.start_date).startOf('month');
        var lastDate = Vue.moment(this.start_date).endOf('month');

        this.selective_working_hour_schedule.push({
          date: firstDate.clone().format('YYYY-MM-DD'),
          week: firstDate.clone().isoWeek(),
          start_time: null,
          end_time: null,
          holiday: false,
          work_min: 0,
          break_min: 0
        })

        while (firstDate.add(1, 'days').diff(lastDate) <= 0) {
          this.selective_working_hour_schedule.push({
            date: firstDate.clone().format('YYYY-MM-DD'),
            week: firstDate.clone().isoWeek(),
            start_time: null,
            end_time: null,
            holiday: false,
            work_min: 0,
            break_min: 0
          })
        }
      }
    }
  }
}
</script>

<style>
.week-table {
  width: 100%;
  border-style:hidden;
}
</style>