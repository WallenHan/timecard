<script setup>
import {ref, onMounted, computed} from "vue";
import dayjs from "dayjs";
import customParseFormat from "dayjs/plugin/customParseFormat";
import utc from "dayjs/plugin/utc";
import timezone from "dayjs/plugin/timezone";

dayjs.extend(customParseFormat);
dayjs.extend(utc);
dayjs.extend(timezone);
// 设置全局时区为东八区
const defaultTimezone = 'Asia/Shanghai';
dayjs.tz.setDefault(defaultTimezone);

const props = defineProps({
  dateList: {
    type: Array,
    required: true,
    validator: (value) => {
      return value.every((item) => {
        // 校验 date 字段是否为有效的日期格式（YYYYMMDD）
        const dateRegex = /^\d{8}$/;
        if (!dateRegex.test(item.date)) {
          return false;
        }
        // 校验 info 字段是否为对象
        if (!(typeof item.info === 'object')){
          return false;
        }
        if (!Array.isArray(item.tasks)){
          return false;
        }
      })
    },
    default: () =>[]
  }
});
let years = ref([]);
const months = ref([`Jan`, `Feb`, `Mar`, `Apr`, `May`, `Jun`, `Jul`, `Aug`, `Sep`, `Oct`, `Nov`, `Dec`]);
const weeks = ref([`Sun`, `Mon`, `Tue`, `Wed`, `Thu`, `Fri`, `Sat`]);
let currentYear =  ref(null);
let currentMonth = ref(null);
let currentDay = ref(null);
let selectedDay = ref(null);

// 当前展示的月份需要展示的天数
let daysOfMonth = computed(() => {
  let  days = Array.from(
      Array(dayjs().year(currentYear.value).month(currentMonth.value).daysInMonth()).keys())
      .map(it => ({date: dayjs().year(currentYear.value).month(currentMonth.value).date(it+1), isThisMonth: true}));
  calcFistDayInMonthIsSunday(days);
  //  根据传入的数据格式化成 表格组件需要的数据
  days = days.map((it,idx) => {
    let {dateList} = props;
    for (let i = 0; i < dateList.length; i++) {
      // 判断时间是否相等
      let dateFromList = dayjs(dateList[i].date + "","YYYYMMDD");
      if(it.date.isSame(dateFromList, 'day')) {
        return {
          date: it.date,
          monthDay: `${it.date.month()+1}-${it.date.date()}`,
          message: dateList[i].message,
          task: dateList[i].task,
          isThisMonth: it.isThisMonth,
          dayType: idx%7 ===0 || idx%7 === 6 ? 0 : 1,
        };
      }
    }
    return {
      date: it.date,
      monthDay: `${it.date.month()+1}-${it.date.date()}`,
      message: null,
      task: [],
      dayType: idx%7 ===0 || idx%7 === 6 ? 0 : 1,
      isThisMonth: it.isThisMonth,
    }

  })
  return days;
});

onMounted(() => {
  const initDate = dayjs();
  const yearList = []
  yearList.push(initDate.year());
  // 扩展前10年
  for (let i = 1; i < 11; i++) {
    yearList.unshift(initDate.year() - i);
  }
  // 扩展后10年
  for (let i = 1; i < 11; i++) {
    yearList.push(initDate.year() + i);
  }
  // 初始化当前的年月日
  years.value = yearList;
  currentYear.value = initDate.year();
  currentMonth.value = initDate.month();
  currentDay.value = initDate.date();
  // 默认选中的是当前天
  selectedDay.value = initDate;
})

/**
 *
 *  补全展示完整月的方法，当该月第一天不是周天时，需要补足上个月的日期,结束没完时补足下个月的
 *  @param days 当月需要展示的天数list
 */
const calcFistDayInMonthIsSunday = (days) => {
  let firstDayInMonth = days[0].date.date(1);
  let firstDayInWeek = firstDayInMonth.day();
  for (let i = 1; i <= firstDayInWeek; i++) {
    let theDay = firstDayInMonth.subtract(i, 'day');
    days.unshift({date: theDay, isThisMonth: false});
  }
  for (let i = days.length, j = 1; i < 42; i++, j++) {
    days.push({date: dayjs().year(currentYear.value).month(currentMonth.value + 1 ).date(j), isThisMonth: false});
  }
}

const judgeIsToday = (date) => {
  return dayjs().isSame(date, 'day');
}

const judeIsSelectedDay = (date) => {
  return selectedDay.value?.isSame(date, 'day');
}

const emit = defineEmits(['dateInfo']);
/**
 *  捕捉点击事件，设置选中的天并抛事件出去
 */
const handleClickDay = (date) => {
  //  非本月的不允许点击
  if(date.isThisMonth){
    selectedDay.value = date.date;
    emit(`dateInfo`, date);
  }
}
</script>

<template>
  <div class="root-wrapper">
    <div class="left-wrapper">
      <select name="year" id="year"  v-model="currentYear" class="select-year">
        <option v-for="year in years" :value="year" :key="year">
          {{ year }}
        </option>
      </select>
      <div class="month-wrapper">
        <div v-for="(month, idx) in months" class="month"
             :class="{'is-checked': currentMonth === idx}"
        :key="`${month}-${idx}`">
          <input type="radio" :id="month" :value="idx" v-model="currentMonth" />
          <label :for="month">{{month}}</label>
        </div>
      </div>
    </div>
    <div class="right-wrapper">
      <div class="right-title">
        <div v-for="week in weeks"  class="week-wrapper" :key="week" >
          {{week}}
        </div>
      </div>
      <div class="right-card">
        <div v-for="(day, idx) in daysOfMonth"
             :key="`${idx}-${day}`"
             :data-date="day.date"
             class="day-wrapper"
             :class="[day.dayType === 0  ? `weekend` : `normal`,
             {'not-this-month': day.isThisMonth === false,
             'is-today': judgeIsToday(day.date) === true,
             'is-selected': judeIsSelectedDay(day.date) === true,}]"
        @click="handleClickDay(day)">
              {{day.monthDay}}
          <div class="task-list">
              <div v-for="(task,idx) in day.task">
                <i>⚫</i> {{task}}
              </div>
          </div>
          <div class="info-wrapper">
            <div v-if="day.message" :class="day.message?.type === 'warn' ? `warn`: `info`" >
                {{day.message?.info}}
            </div>
          </div>
          <div class="day-type">
              {{day.dayType === 0  ? `周末`: `工作日`}}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>

.root-wrapper {
  --border-color: #acaaaa;
  --today-border-color: #67B160;

  --weekend-bg-color: #99EA5F;
  --workday-bg-color: #fff;
  --info-bg-color: #67B160;
  --warning-bg-color: #3dc321;
  --error-bg-color: #b81a1a;
  --selected-bg-color: #E14F52;

  --info-color: #fff;
  --warning-color: #fff;
  --selected-color: #fff;


  --day-width: 8rem;
  --day-height: 7rem;
  --week-gap-size: 1rem;
  --border-radius: 2px;

  display: grid;
  grid-template-columns: 1fr 7fr;
  min-height: 45rem;
  grid-column-gap: 1rem;

  .select-year {
    width: 6rem;
    font-size: 1.2rem;
    font-weight: bold;
  }
  .month-wrapper {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    margin-top: 2rem;
    .month {
      border: var(--border-color) solid 1px;
      padding: .5rem 1rem;
      margin-right: -1px;
      margin-bottom: -1px;
      input {
        display: none;
      }
      label {
        display: inline-block;
        width: 100%;
        height: 100%;
      }
    }
    .is-checked {
      background-color: var(--selected-bg-color);
      label {
        color: var(--selected-color);
      }
    }

  }
  .right-wrapper {


    display: grid;
    grid-template-rows: 2rem 1fr;
    grid-row-gap: 1rem;

    .right-title {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      grid-gap: var(--week-gap-size);
      border-bottom: var(--border-color) solid 1px;

      .week-wrapper {
        font-weight: bolder;
        font-size: 1.2rem;
        text-align: left;
        padding-left: 1rem;
        cursor: pointer;
      }
    }
    .right-card {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      grid-gap: var(--week-gap-size);

      .day-wrapper {
        width: var(--day-width);
        height: var(--day-height);
        position: relative;
        text-align: left;
        padding-left: 1rem;
        border: 1px solid var(--border-color);
        border-radius: var(--border-radius);
        cursor: pointer;

        .day-type {
          position: absolute;
          right: 0;
          bottom: 0;
          padding: .25rem .5rem;
          border: 1px solid var(--border-color);
          margin-right: -1px;
          margin-bottom: -1px;
          font-size: .8rem;
        }
        .task-list {
          margin-top: 1rem;
          font-size: .6rem;

          i {
            font-size: .4rem;
            vertical-align: text-top;
          }
        }
        .info-wrapper {
          position: absolute;
          top: .25rem;
          right: .5rem;
          .info {
            background: var(--info-bg-color);
            color: var(--info-color);
          }
          .warn {
            background: var(--warning-bg-color);
            color: var(--warning-color);
          }
          .info,.warn {
            font-size: .6rem;
            width: 3rem;
            padding: .1rem .2rem;
          }
        }

      }
      .weekend {
        background: var(--weekend-bg-color);
        .day-type {
          border: 0;
        }
      }
      .is-today {
        outline: var(--today-border-color) solid 1px;
      }
      .is-selected {
        outline: var(--selected-bg-color) solid 1px;
      }
      .normal {
        background: var(--workday-bg-color);
      }
      .not-this-month {
        opacity: .7;
        cursor: not-allowed;
      }

    }
  }

}

</style>