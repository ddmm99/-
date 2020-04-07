<template>
  <div class="date-picker" v-click-outside>
    <div class="picker-input">
      <span class="input-prefix">
        <i class="iconfont icon-date"></i>
      </span>
      <input type="text" :value="chooseDate" />
    </div>
    <div class="picker-paner" v-if="showPanel">
      <div class="picker-arrow" />
      <div class="picker-body">
        <div class="picker-header">
          <div class="picker-btn icon-prev-year" @click="onChangeYear('prev')" />
          <div class="picker-btn icon-prev-month" @click="onChangeMonth('prev')"/>
          <span class="picker-date">{{ showDate.year }} 年 {{ showDate.month + 1 }} 月</span>
          <div class="picker-btn icon-next-month"  @click="onChangeMonth('next')" />
          <div class="picker-btn icon-next-year" @click="onChangeYear('next')" />
        </div>
        <div class="picker-content">
          <div class="picker-weeks">
            <div v-for="week in ['日','一','二','三','四','五','六']" :key="week">{{week}}</div>
          </div>
          <div class="picker-days">
            <div
              class
              v-for="date in showDay"
              :key="date.getTime()"
              :class="{'other-month':!isCur(date).month,
                         'is-select': isCur(date).select,
                         'is-today':!isCur(date).today}"
              @click="onChooseDate(date)"
            >{{ date.getDate(date) }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  model:{
      prop:'date',
      event:'chooseDate'
  },
  directives: {
    "click-outside": {
      bind(el, binding, vnode) {
        const vm = vnode.context;
        document.onclick = function(e) {
          const dom = e.target;
          const isElSon = el.contains(dom);
          if (isElSon && !vm.showPanel) {
            vm.changePanel(true);
          } else if (!isElSon && vm.showPanel) {
            vm.changePanel(false);
          }
        };
      }
    }
  },
  props: {
    date: {
      type: Date,
      default: () => new Date()
    }
  },

  computed: {
    chooseDate() {
      const { year, month, day } = this.getYearMonthDay(this.date);
      return `${year}-${month + 1}-${day}`;
    },
    showDay() {
      const { year, month } = this.showDate;
      const firstDay = new Date(year, month, 1);
      const week = firstDay.getDay();
      const startDay = firstDay - week * 24 * 60 * 60 * 1000;
      let arr = [];
      for (let i = 0; i < 42; i++) {
        arr.push(new Date(startDay + i * 24 * 60 * 60 * 1000));
      }
      return arr;
    }
  },
  data() {
    return {
      showPanel: false,
      showDate: {
        year: 0,
        month: 0,
        day: 0
      }
    };
  },
  created() {
    this.getShowDate(this.date);
  },
  methods: {
    onChangeYear(type){
        const moveYear = type === 'prev' ? -1 : 1;
        this.showDate.year += moveYear;
    },
    onChangeMonth(type){
        const {year,month,day} = this.showDate;
        const moveMonth = type === 'prev' ? -1 : 1;
        const showDate = new Date(year,month,day);
        showDate.setMonth(month + moveMonth);
        const {year:showYear,month:showMonth} = this.getYearMonthDay(showDate);
        this.showDate.year = showYear;
        this.showDate.month = showMonth; 


    //    const maxMonth = 11;
    //    const minMonth = 0;
    //    let {year,month} = this.showDate;
    //    month += moveMonth;
    //    if(month < minMonth) {
    //        month = maxMonth;
    //        year-- ;
    //    }else if(month > maxMonth){
    //        month = minMonth;
    //        year++ ;
    //    }
    //    this.showDate.month = month; 
    //    this.showDate.year = year;
    },
    changePanel(flag) {
      this.showPanel = flag;
    },
    onChooseDate(date) {
      this.$emit("chooseDate", date);
      this.showPanel = false;
      this.getShowDate(date);
    },
    getShowDate(date) {
      const { year, month, day } = this.getYearMonthDay(date);
      this.showDate = {
        year,
        month,
        day
      };
    },
    getYearMonthDay(date) {
      const year = date.getFullYear();
      const month = date.getMonth();
      const day = date.getDate();
      return { year, month, day };
    },
    isCur(date) {
      const chooseDate = new Date(this.chooseDate);

      const { year: showYear, month: showMonth } = this.showDate;
      const {
        year: chooseYear,
        month: chooseMonth,
        day: chooseDay
      } = this.getYearMonthDay(chooseDate);
      const {
        year: curYear,
        month: curMonth,
        day: curDay
      } = this.getYearMonthDay(date);
      const { year, month, day } = this.getYearMonthDay(date);
      return {
        month: year === showYear && month === showMonth,
        select:
          year === chooseYear && month === chooseMonth && chooseDay === day,
        today: year === curYear && month === curMonth && day === curDay
      };
    }
  }
};
</script>


<style scoped>
@import url("./assets/font.css");
.date-picker {
  display: inline-block;
}
.picker-input {
  position: relative;
}
.picker-input input {
  height: 40px;
  line-height: 40px;
  padding: 0 30px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  outline: none;
  cursor: pointer;
  background-color: #ffffff;
}

.picker-input .input-prefix {
  color: #c0c4cc;
  width: 25px;
  position: absolute;
  left: 5px;
  height: 100%;
  line-height: 40px;
  text-align: center;
}
.picker-paner {
  position: absolute;
  width: 322px;
  height: 329px;
  border: 1px solid #e4e7ed;
  border-radius: 4px;
  margin-top: 5px;
  background-color: #ffffff;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}
.picker-paner .picker-arrow {
  position: absolute;
  top: -12px;
  left: 30px;
  width: 0;
  height: 0;
  border: 6px solid transparent;
  border-bottom-color: #dcdfe6;
}
.picker-paner .picker-arrow::before {
  content: "";
  position: absolute;
  top: -4px;
  left: -5px;
  width: 0;
  height: 0;
  border: 5px solid transparent;
  border-bottom-color: #fff;
}
.picker-paner .picker-header {
  width: 85%;
  margin-left: 50%;
  transform: translateX(-50%);
  align-items: center;
  display: flex;
  justify-content: space-between;
  padding-top: 15px;
  padding-bottom: 10px;
}
.picker-paner .picker-date {
  user-select: none;
}
.picker-paner .picker-btn {
  font-size: 12px;
  color: #303133;
  cursor: pointer;
}
.picker-paner .picker-content {
  padding: 0 10px 10px 10px;
  color: #606266;
  user-select: none;
}
.picker-content .picker-weeks {
  display: flex;
  justify-content: space-around;
  cursor: pointer;
  height: 40px;
  line-height: 40px;
  border-bottom: 1px solid #ebeef5;
}
.picker-content .picker-days {
  display: flex;
  flex-wrap: wrap;
}
.picker-content .picker-days div {
  width: 30px;
  height: 30px;
  line-height: 30px;
  margin: 4px 6px;
  text-align: center;
  cursor: pointer;
  display: flex;
  font-size: 12px;
  justify-content: space-around;
}
.picker-content .picker-days div:hover {
  color: #4091ff;
}
.picker-content .picker-days div.is-today {
  color: #4091ff;
  font-weight: 700;
}
.picker-content .picker-days div.is-select {
  background-color: #4091ff;
  color: #fff;
  border-radius: 50%;
}
.picker-content .picker-days div.other-month {
  color: #c0c4cc;
}
</style>