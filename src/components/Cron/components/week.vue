<template lang="html">
  <div :val="value_">
    <div>
      <el-radio v-model="type" label="1" size="mini" border>{{$t('system.cron', {FIELD: 'per'})}}{{lable}}</el-radio>
    </div>
    <div>
      <el-radio v-model="type" label="5" size="mini" border>{{$t('system.cron', {FIELD: 'noPointAt'})}}</el-radio>
    </div>
    <div>
      <el-radio v-model="type" label="2" size="mini" border>{{$t('system.cron', {FIELD: 'period'})}}</el-radio>
      <span style="margin-left: 10px; margin-right: 5px;">{{$t('system.cron', {FIELD: 'from'})}}{{$t('system.cron', {FIELD: 'weekday'})}}</span>
      <!-- <el-input-number controls-position="right" @change="type = '2'" v-model="cycle.start" :min="1" :max="7" size="mini" style="width: 100px;"></el-input-number> -->
      <el-select v-model="cycle.start" style="width: 100px;" clearable @change="type = '2'">
        <el-option
          v-for="(i, v) in weeks"
          :key="'key'+i"
          :label="i"
          :value="''+ (v+1)">
        </el-option>
      </el-select>
      <span style="margin-left: 5px; margin-right: 5px;">{{$t('system.cron', {FIELD: 'to'})}}{{$t('system.cron', {FIELD: 'weekday'})}}</span>
      <!-- <el-input-number controls-position="right" @change="type = '2'" v-model="cycle.end" :min="2" :max="7" size="mini" style="width: 100px;"></el-input-number> -->
      <el-select v-model="cycle.end" style="width: 100px;" clearable @change="type = '2'">
        <el-option
          v-for="(i, v) in weeks"
          :key="'key'+i"
          :label="i"
          :value="''+ (v+1)">
        </el-option>
      </el-select>
    </div>
    <!-- <div>
      <el-radio v-model="type" label="3" size="mini" border>循环</el-radio>
      <span style="margin-left: 10px; margin-right: 5px;">从星期</span>
      <el-input-number controls-position="right" @change="type = '3'" v-model="loop.start" :min="1" :max="7" size="mini" style="width: 100px;"></el-input-number>
      <span style="margin-left: 5px; margin-right: 5px;">开始，每</span>
      <el-input-number controls-position="right" @change="type = '3'" v-model="loop.end" :min="1" :max="7" size="mini" style="width: 100px;"></el-input-number>
      天执行一次
    </div> -->
    <!-- <div>
      <el-radio v-model="type" label="7" size="mini" border>指定周</el-radio>
      <span style="margin-left: 10px; margin-right: 5px;">本月第</span>
      <el-input-number controls-position="right" @change="type = '7'" v-model="week.start" :min="1" :max="4" size="mini" style="width: 100px;"></el-input-number>
      <span style="margin-left: 5px; margin-right: 5px;">周，星期</span>
      <el-input-number controls-position="right" @change="type = '7'" v-model="week.end" :min="1" :max="7" size="mini" style="width: 100px;"></el-input-number>
    </div> -->
    <!-- <div>
      <el-radio v-model="type" label="6" size="mini" border>本月最后一个</el-radio>
      <span style="margin-left: 10px; margin-right: 5px;">星期</span>
      <el-input-number controls-position="right" @change="type = '6'" v-model="last" :min="1" :max="7" size="mini" style="width: 100px;"></el-input-number>
    </div> -->
    <div>
      <el-radio v-model="type" label="4" size="mini" border>{{$t('system.cron', {FIELD: 'pointAt'})}}</el-radio>
      <el-checkbox-group v-model="appoint" style="margin-left: 50px;  line-height: 25px;">
          <el-checkbox @change="type = '4'"  v-for="i in 7" :key="i" :label="'' + i">{{weeks[i - 1]}}</el-checkbox>
      </el-checkbox-group>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: '?'
    },
    lable: {
      type: String
    }
  },
  data () {
    return {
      type: '5', // 类型
      cycle: { // 周期
        start: '2',
        end: '2'
      },
      loop: { // 循环
        start: 0,
        end: 0
      },
      week: { // 指定周
        start: 0,
        end: 0
      },
      work: 0,
      last: 0,
      appoint: [], // 指定
      weeks: ['日', '一', '二', '三', '四', '五', '六'],
    }
  },
  computed: {
    value_ () {
      const result = []
      switch (this.type) {
        case '1': // 每秒
          result.push('*')
          break
        case '2': // 年期
          result.push(`${this.cycle.start}-${this.cycle.end}`)
          break
        case '3': // 循环
          result.push(`${this.loop.start}/${this.loop.end}`)
          break
        case '4': // 指定
          // result.push(this.appoint.join(','))
          result.push(this.appoint.length ? this.appoint.join(',') : '1')
          break
        case '6': // 最后
          result.push(`${this.last === 0 ? '' : this.last}L`)
          break
        case '7': // 指定周
          result.push(`${this.week.start}#${this.week.end}`)
          break
        default: // 不指定
          result.push('?')
          break
      }
      this.$emit('input', result.join(''))
      return result.join('')
    }
  },
  watch: {
    'value' (a, b) {
      this.updateVal()
    },
    type (val) {
      if (val !== '4') {
        this.appoint = []
      }
    }
  },
  methods: {
    updateVal () {
      if (!this.value) {
        return
      }
      if (this.value === '?') {
        this.type = '5'
      } else if (this.value.indexOf('-') !== -1) { // 2周期
        if (this.value.split('-').length === 2) {
          this.type = '2'
          this.cycle.start = this.value.split('-')[0]
          this.cycle.end = this.value.split('-')[1]
        }
      } else if (this.value.indexOf('/') !== -1) { // 3循环
        if (this.value.split('/').length === 2) {
          this.type = '3'
          this.loop.start = this.value.split('/')[0]
          this.loop.end = this.value.split('/')[1]
        }
      } else if (this.value.indexOf('*') !== -1) { // 1每
        this.type = '1'
      } else if (this.value.indexOf('L') !== -1) { // 6最后
        this.type = '6'
        this.last = this.value.replace('L', '')
      } else if (this.value.indexOf('#') !== -1) { // 7指定周
        if (this.value.split('#').length === 2) {
          this.type = '7'
          this.week.start = this.value.split('#')[0]
          this.week.end = this.value.split('#')[1]
        }
      } else if (this.value.indexOf('W') !== -1) { // 8工作日
        this.type = '8'
        this.work = this.value.replace('W', '')
      } else { // *
        this.type = '4'
        this.appoint = this.value.split(',')
      }
    }
  },
  created () {
    this.updateVal()
  }
}
</script>

<style lang="css">
.el-checkbox+.el-checkbox {
  margin-left: 10px;
}
</style>
