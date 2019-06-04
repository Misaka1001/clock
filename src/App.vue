<template>
  <div id="app">
    <div id="clock" :style="`transform:scale(${scale})`">
      <svg width="500" height="500">
      <g class="circle">
        <circle cx="250" cy="250" r="150" stroke="rgb(50,95,162)" fill="none" stroke-width="15"/>
        <line v-for="val in icon_1" :key="val" :transform="`translate(250,250)rotate(${val})`" x1="0" y1="-120" x2="0" y2="-135" style="stroke:#333;stroke-width:6;stroke-linecap:round"/>
        <line v-for="val in icon_2" :key="val" :transform="`translate(250,250)rotate(${val})`" x1="0" y1="-130" x2="0" y2="-135" style="stroke:#333;stroke-width:4;stroke-linecap:round"/>
      </g>
      <g class="hour" @mousedown="drag_change('hour')"
      :transform="`translate(250,250)rotate(${hour * 30 + minute * 0.5})`">
        <line x1="0" y1="15" x2="0" y2="-100" style="stroke:#333;stroke-width:15;stroke-linecap:round"/>
      </g>
      <g class="minute" @mousedown="drag_change('minute')"
        :transform="`translate(250,250)rotate(${minute * 6})`">
        <line x1="0" y1="15" x2="0" y2="-130" style="stroke:#333;stroke-width:10;stroke-linecap:round"/>
      </g>
      <g class="second" @mousedown="drag_change('second')"
        :transform="`translate(250,250)rotate(${second * 6})`">
        <line x1="0" y1="15" x2="0" y2="-100" style="stroke:#d00;stroke-width:8;stroke-linecap:round"/>
        <circle cx="0" cy="-108" r="8" fill="none" style="stroke:#d00;stroke-width:7"/>
      </g>
      <circle cx="250" cy="250" r="10" stroke="#d00" fill="#d00"/>
    </svg>
    </div>
    <div class="control">
      <section>
        <button @click="hourBtn(true)">+</button>
        <button @click="minuteBtn(true)">+</button>
        <button @click="secondBtn(true)">+</button>
      </section>
      <section class="time">
        <span>{{ hour > 9 ? Math.floor(hour) : '0' + Math.floor(hour) }}</span>
        <span>:</span>
        <span>{{ minute > 9 ? Math.floor(minute) : '0' + Math.floor(minute) }}</span>
        <span>:</span>
        <span>{{ second > 9 ? Math.floor(second) : '0' + Math.floor(second) }}</span>
      </section>
      <section>
        <button @click="hourBtn(false)">-</button>
        <button @click="minuteBtn(false)">-</button>
        <button @click="secondBtn(false)">-</button>
      </section>
    </div>
  </div>
</template>
<script>
export default {
  mounted () {
    this.init()
    window.onresize = () => {
      this.vw = document.body.clientWidth
    }
  },
  data () {
    return {
      icon_1: (function () {
        let arr = []
        for (let i = 0; i <= 12; i++) {
          arr.push(i * 30)
        }
        return arr
      }()),
      icon_2: (function () {
        let arr = []
        for (let i = 1; i < 60; i++) {
          if (i * 6 % 30 !== 0) {
            arr.push(i * 6)
          }
        }
        return arr
      }()),
      interval: null,
      vw: document.body.clientWidth,
      time: Date.now(),
      hour: 0,
      minute: 0,
      second: 0,
      hour_add: 0,
      minute_add: 0,
      second_add: 0
    }
  },
  methods: {
    init () {
      this.time = Date.now()
      this.interval = setInterval(() => {
        this.time = Date.now()
      }, 1000)
    },
    hourBtn (flag) {
      clearInterval(this.interval)
      if (flag) {
        this.hour_add += 3600000
        if (this.hour > 12) {
          this.hour_add = -new Date(this.time).getHours * 3600000
        }
      } else {
        this.hour_add -= 3600000
        if (this.hour < 0) {
          this.hour_add = 23 - new Date(this.time).getHours * 3600000
        }
      }
      this.init()
    },
    minuteBtn (flag) {
      clearInterval(this.interval)
      if (flag) {
        this.minute_add += 60000
        if (this.minute === 60) {
          this.minute_add = -this.time.getMinutes()
          this.hour_add += 3600000
        }
      } else {
        this.minute_add -= 60000
        if (this.minute < 0) {
          this.minute_add = 59 - this.time.getMinutes()
          this.hour_add -= 3600000
        }
      }
      this.init()
    },
    secondBtn (flag) {
      clearInterval(this.interval)
      if (flag) {
        this.second_add += 1000
        if (this.second >= 60) {
          this.second_add = -this.time.getSeconds() * 1000
          this.minute_add += 60000
        }
      } else {
        this.second_add -= 1000
        if (this.second < 0) {
          this.second_add = 59 - this.time.getSeconds() * 1000
          this.minute_add -= 60000
        }
      }
      this.init()
    },
    drag_change (prop) {
      document.addEventListener('mousemove', move, false)
      clearInterval(this.interval)
      let self = this
      function move (e) {
        let clock = document.getElementById('clock')
        let left = clock.offsetLeft
        let width = clock.offsetWidth
        let height = clock.offsetHeight
        let x = e.clientX - left - width / 2
        let y = height / 2 - e.clientY
        let rotate = 0
        if (x >= 0) {
          if (y >= 0) {
            rotate = Math.atan(x / y) * 180 / Math.PI
          } else {
            rotate = Math.atan(Math.abs(y) / x) * 180 / Math.PI + 90
          }
        } else {
          if (y >= 0) {
            rotate = Math.atan(y / Math.abs(x)) * 180 / Math.PI + 270
          } else {
            rotate = Math.atan(Math.abs(x) / Math.abs(y)) * 180 / Math.PI + 180
          }
        }
        let now = new Date(self.time)
        if (prop === 'hour') {
          let oldHour = now.getHours()
          if (oldHour >= 12) {
            oldHour -= 12
          }
          self.hour_add = (rotate / 30 - 1) * 3600000 - oldHour * 3600000
        } else if (prop === 'minute') {
          let oldminute = now.getMinutes()
          self.minute_add = (rotate / 6) * 60000 - oldminute * 60000
        } else if (prop === 'second') {
          let oldSecond = now.getSeconds()
          self.second_add = (rotate / 6) * 1000 - oldSecond * 1000
        }
      }
      function up () {
        document.removeEventListener('mousemove', move)
        document.removeEventListener('mouseup', up)
        self.init()
      }
      document.addEventListener('mouseup', up, false)
    },
    watch_time () {
      let now = new Date(this.time + this.time_add)
      this.hour = (function () {
        let hour = now.getHours()
        if (hour >= 13) {
          return hour - 12
        } else if (hour === 0) {
          return 12
        } else {
          return hour
        }
      }())
      this.minute = now.getMinutes()
      this.second = now.getSeconds()
    }
  },
  watch: {
    time () {
      this.watch_time()
    },
    time_add () {
      this.watch_time()
    }
  },
  computed: {
    scale () {
      let vw = this.vw
      if (vw >= 1000) {
        return vw * 0.3 / 500
      } else if (vw < 1000 && vw >= 800) {
        return vw * 0.4 / 500
      } else if (vw < 800 && vw >= 480) {
        return vw * 0.6 / 500
      } else if (vw < 480) {
        return vw * 0.8 / 500
      }
    },
    time_add () {
      return this.hour_add + this.minute_add + this.second_add
    }
  }
}
</script>
<style lang="scss">
* {
  padding: 0;
  margin: 0;
}
:root {
  font-size: 1vw;
  display: flex;
  justify-content: center;
  align-items: center;
}
@media screen and(max-width : 800px){
  :root{
    font-size: 2vw;
  }
}
body{
  width: 100%;
  height: 100%;
  #app {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    width: 100%;
    #clock{
      width: 500px;
      height: 500px;
    }
    .control{
      width: 30rem;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      button{
        font-size: 3rem;
        width: 7rem;
        height: 3rem;
        line-height: 2rem;
        border-radius: 0.1rem;
        outline: none;
        background: #fff;
        border: 1px solid #333;
        border-radius: 0.5rem;
        margin: 0 0.3rem;
        &:active{
          background: rgba(0, 0, 0, 0.1);
        }
      }
      .time{
        font-size: 2rem;
        height: 3rem;
        line-height: 3rem;
        span:nth-of-type(2n+1){
          display: inline-block;
          width: 7rem;
          text-align: center;
        }
        span:nth-of-type(2n){
          display: inline-block;
          width: 0.3rem;
          text-align: center;
        }
      }
    }
  }
}
</style>
