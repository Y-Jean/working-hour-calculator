<template>
  <div class="row">
    <div class="title mx-auto">
        <h1>근로 시간 계산기</h1>
    </div>
    <div class="col-12 mt-5">
      <div class="p-2">
        <div class="form-group d-flex align-items-center">
          <div class="font-weight-bold">근무제 종류</div>
          <div class="ml-4">
            <select v-model="workingSelect" class="form-control w-300px" @change="changeWorking($event.target.value)">
              <option v-for="working in workings" :key="working.value" :value="working.value">
                {{ working.text }}
              </option>
            </select>
          </div>
          <div class="ml-5 font-weight-bold">단위 기간</div>
          <div class="ml-4">
            <select v-model="periodSelect" class="form-control">
              <option v-for="period in periods" :key="period.id" :value="period">
                {{ period.amount
                }}<span v-if="period.unit === 'week'">주</span
                ><span v-else>개월</span>
              </option>
            </select>
          </div>
        </div>
        <template v-if="periodSelect !== null">
          <component
            :is="workingSelect"
            :period="periodSelect"
            class="mt-2"
          ></component>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import W001 from '~/components/policies/W001'
import W002 from '~/components/policies/W002'
import W003 from '~/components/policies/W003'
import W004 from '~/components/policies/W004'

export default {
  components: {
    W001,
    W002,
    W003,
    W004
  },
  data() {
    return {
      workingSelect: '',
      workings: [
        { text: '일반근무제', value: 'W001', period: [ {id: '1', unit: 'week', amount: 1} ] },
        { text: '시차출퇴근제', value: 'W002', period: [ {id: '1', unit: 'week', amount: 1} ] },
        { text: '선택적 근로시간제', value: 'W003', period: [
          {id: '1', unit: 'week', amount: 1},
          {id: '2', unit: 'week', amount: 2},
          {id: '3', unit: 'week', amount: 3},
          {id: '4', unit: 'week', amount: 4},
          {id: '5', unit: 'month', amount: 1}
        ] },
        { text: '탄력적 근로시간제', value: 'W004', period: [
          {id: '2', unit: 'week', amount: 2},
          {id: '3', unit: 'week', amount: 3},
          {id: '4', unit: 'week', amount: 4},
          {id: '5', unit: 'week', amount: 5},
          {id: '6', unit: 'week', amount: 6},
          {id: '7', unit: 'week', amount: 7},
          {id: '8', unit: 'week', amount: 8},
          {id: '9', unit: 'week', amount: 9},
          {id: '10', unit: 'week', amount: 10},
          {id: '11', unit: 'week', amount: 11},
          {id: '12', unit: 'week', amount: 12},
        ] }
      ],
      periods: [],
      periodSelect: null,
    }
  },
  watch: {
    'workingSelect': {
      handler: function() {
        this.changePeriod()
      }
    }
  },
  mounted() {
    this.workingSelect = 'W001',
    this.periodSelect = this.changePeriod()
  },
  methods: {
    changeWorking(type) {
      let data = _.find(this.workings, { 'value': type })
      this.periods = data.period
    },
    changePeriod() {
      this.periods = _.find(this.workings, {'value': this.workingSelect}).period

      if (this.workingSelect !== 'W003' && this.workingSelect !== 'W004') {
        this.periodSelect = this.periods[0]
      } else {
        this.periodSelect = null
      }
    }
  }
}
</script>