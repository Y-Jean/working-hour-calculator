<template>
  <div>
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
              :class="[disableCheck(key) ? 'disabled' : '', controlData.day[key] ? 'active' : '']"
            >
              <input
                v-model="controlData.day[key]"
                type="checkbox"
                class="custom-control-input"
                :disabled="disableCheck(key)"
              />
              {{ dayname(key) }}
            </label>
          </div>
        </div>
        <div class="d-flex align-items-center mt-2">
          <label class="mr-3">시간</label>
          <select
            v-model="controlData.timeType"
            class="form-control w-180px mr-3"
          >
            <option value="work">근로시간</option>
            <option value="break">휴게시간대</option>
          </select>
          <div class="wrap-timepicker">
            <template v-if="controlData.timeType === 'work'">
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
            <template v-else>
              <VueCtkDateTimePicker
                id="break_time"
                v-model="controlData.break_time"
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
              <span class="mx-1"> ~ </span>
              <VueCtkDateTimePicker
                id="to_break_time"
                v-model="controlData.to_break_time"
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
    <div class="mt-3">※ 토요일/일요일은 휴일입니다.</div>

    <div class="mt-3">
      <p class="font-weight-bold mb-2">
        스케줄
      </p>
      <table class="table text-center small">
        <caption>&middot; 해당 요일이 공휴일이라면 '공휴일여부'에 표기해주세요.</caption>
        <thead class="font-secondary">
          <tr>
            <th scope="col" class="w-100px">
              요일
            </th>
            <th scope="col">
              출근
            </th>
            <th scope="col">
              퇴근
            </th>
            <th scope="col">
              휴게시간대
            </th>
            <th scope="col">
              공휴일여부
            </th>
            <th scope="col">
              근로시간
            </th>
            <th scope="col">
              휴게시간
            </th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(schedule, key) in weekly_schedule" :key="key">
            <td scope="row">{{ dayname(key) }}요일</td>
            <td>
              <div class="d-flex align-items-center justify-content-center">
                <span><VueCtkDateTimePicker
                    :id="key + '-start_time'"
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
                    :id="key + '-end_time'"
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
              <div class="d-flex align-items-center justify-content-center">
                <span
                  ><VueCtkDateTimePicker
                    :id="key + '-break_time'"
                    v-model="schedule.break_time"
                    :disabled="schedule.holiday === true"
                    :no-label="true"
                    label=""
                    format="HH:mm"
                    formatted="HH:mm"
                    :minute-interval="10"
                    input-size="sm"
                    :only-time="true"
                    class="only-time"
                  >
                  </VueCtkDateTimePicker
                ></span>
                <span class="mx-1">~</span>
                <span
                  ><VueCtkDateTimePicker
                    :id="key + '-to_break_time'"
                    v-model="schedule.to_break_time"
                    :disabled="schedule.holiday === true"
                    :no-label="true"
                    label=""
                    format="HH:mm"
                    formatted="HH:mm"
                    :minute-interval="10"
                    input-size="sm"
                    :only-time="true"
                    class="only-time"
                  >
                  </VueCtkDateTimePicker
                ></span>
              </div>
            </td>
            <td>
              <div class="align-items-center form-check" v-if="key !== 'sat' && key !== 'sun'">
                <input v-model="schedule.holiday" class="form-check-input" type="checkbox" value="" :id="key + 'holiday'">
                <label class="form-check-label" :for="key + 'holiday'">공휴일</label>
              </div>
            </td>
            <td>{{ minuteDisplay(dailyCalc(key)) }}</td>
            <td>
              {{ minuteDisplay(schedule.break_min) }}
            </td>
          </tr>

          <tr class="alert-primary font-weight-bold border-bottom">
            <td colspan="7">
              1주
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="d-flex align-items-center">
      <div class="font-weight-bold">총 근로시간 : </div>
      <div class="font-weight-bold ml-2">{{ minuteDisplay(total_worktime()) }}</div>
    </div>
    <div class="d-flex align-items-center mt-2" v-if="this.total_time > 2400">
      <div>( 소정 근로시간 : 40시간</div>
      <div class="ml-3 mr-3">/</div>
      <div>추가 근로시간 : {{minuteDisplay(this.total_time - 2400)}} )</div>
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
  data() {
    return {
      controlData: {
        type: 'day',
        timeType: 'work',
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
      minDate: Vue.moment()
        .day(8)
        .format('YYYY-MM-DD'),
      daytitle: {
        mon: '월',
        tue: '화',
        wed: '수',
        thu: '목',
        fri: '금',
        sat: '토',
        sun: '일'
      },
      weekly_schedule: {
        mon: {
          start_time: null,
          end_time: null,
          break_time: null,
          to_break_time: null,
          holiday: false,
          work_min: 0,
          break_min: 0
        },
        tue: {
          start_time: null,
          end_time: null,
          break_time: null,
          to_break_time: null,
          holiday: false,
          work_min: 0,
          break_min: 0
        },
        wed: {
          start_time: null,
          end_time: null,
          break_time: null,
          to_break_time: null,
          holiday: false,
          work_min: 0,
          break_min: 0
        },
        thu: {
          start_time: null,
          end_time: null,
          break_time: null,
          to_break_time: null,
          holiday: false,
          work_min: 0,
          break_min: 0
        },
        fri: {
          start_time: null,
          end_time: null,
          break_time: null,
          to_break_time: null,
          holiday: false,
          work_min: 0,
          break_min: 0
        },
        sat: {
          start_time: null,
          end_time: null,
          break_time: null,
          to_break_time: null,
          holiday: true,
          work_min: 0,
          break_min: 0
        },
        sun: {
          start_time: null,
          end_time: null,
          break_time: null,
          to_break_time: null,
          holiday: true,
          work_min: 0,
          break_min: 0
        }
      },
      total_time: 0
    }
  },
  methods: {
    disableCheck(key) {
      return this.weekly_schedule[key].holiday === true
    },
    dayname(val) {
      return this.daytitle[val]
    },
    total_worktime() {
      let time = 0

      window._.forEach(this.weekly_schedule, val => {
        if (val.work_min !== null) {
          time += val.work_min
        }
      })

      return this.total_time = time
    },
    dailyCalc(key) {
      if (this.weekly_schedule[key].start_time !== null && this.weekly_schedule[key].end_time !== null && this.weekly_schedule[key].holiday === false) {
        let start_time = Vue.moment(this.weekly_schedule[key].start_time, 'HH:mm')
        let end_time = Vue.moment(this.weekly_schedule[key].end_time, 'HH:mm')
        let work_time = 0
        let break_time = 0

        if (start_time > end_time) {
          const start_target = Vue.moment('24:00', 'HH:mm')
          let start_cal = start_target.diff(start_time, 'minutes')

          const end_target = Vue.moment('00:00', 'HH:mm')
          let end_cal = end_time.diff(end_target, 'minutes')

          work_time = start_cal + end_cal
        } else {
          work_time = end_time.diff(start_time, 'minutes')
        }

        if (this.weekly_schedule[key].break_time !== null && this.weekly_schedule[key].to_break_time !== null) {
          let start_break_time = Vue.moment(this.weekly_schedule[key].break_time, 'HH:mm')
          let end_break_time = Vue.moment(this.weekly_schedule[key].to_break_time, 'HH:mm')

          break_time = end_break_time.diff(start_break_time, 'minutes')
        }

        if (break_time > 0 && work_time > break_time) {
          this.weekly_schedule[key].work_min = work_time - break_time
          this.weekly_schedule[key].break_min = break_time
        } else {
          this.weekly_schedule[key].work_min = work_time
          this.weekly_schedule[key].break_min = 0
        }
      } else if (this.weekly_schedule[key].holiday === false) {
        this.weekly_schedule[key].work_min = 0
        this.weekly_schedule[key].break_min = 0
      } else if(this.weekly_schedule[key].holiday === true && key !== 'sat' && key !== 'sun') {
        this.weekly_schedule[key].work_min = 480
        this.weekly_schedule[key].break_min = 0
      }

      return this.weekly_schedule[key].work_min
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

        if (data.timeType === 'work') {
          if (data.start_time !== undefined) {
            this.weekly_schedule[key].start_time = data.start_time
          }

          if (data.end_time !== undefined) {
            this.weekly_schedule[key].end_time = data.end_time
          }
        } else {
          if (data.break_time !== undefined) {
            this.weekly_schedule[key].break_time = data.break_time
          }

          if (data.to_break_time !== undefined) {
            this.weekly_schedule[key].to_break_time = data.to_break_time
          }
        }
      })
    }
  }
}
</script>