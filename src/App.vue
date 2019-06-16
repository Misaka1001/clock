<template>
  <div id="app">
    <div id="clock">
      <svg width="500" height="500" :style="`transform:scale(${scale})`">
      <g class="circle">
        <circle cx="250" cy="250" r="150" stroke="rgb(50,95,162)" fill="none" stroke-width="15"/>
        <line v-for="val in 12" :key="'large' + val" :transform="`translate(250,250)rotate(${val * 30})`" x1="0" y1="-120" x2="0" y2="-135" style="stroke:#333;stroke-width:6;stroke-linecap:round"/>
        <line v-for="val in 60" :key="'little' + val" :transform="`translate(250,250)rotate(${val * 6})`" x1="0" y1="-130" x2="0" y2="-135" style="stroke:#333;stroke-width:4;stroke-linecap:round"/>
      </g>
      <g class="hour" @mousedown="dragChange('hour')"
      :transform="`translate(250,250)rotate(${hour * 30 + minute * 0.5})`">
        <line x1="0" y1="15" x2="0" y2="-100" style="stroke:#333;stroke-width:15;stroke-linecap:round"/>
      </g>
      <g class="minute" @mousedown="dragChange('minute')"
        :transform="`translate(250,250)rotate(${minute * 6})`">
        <line x1="0" y1="15" x2="0" y2="-130" style="stroke:#333;stroke-width:10;stroke-linecap:round"/>
      </g>
      <g class="second" @mousedown="dragChange('second')"
        :transform="`translate(250,250)rotate(${second * 6})`">
        <line x1="0" y1="15" x2="0" y2="-100" style="stroke:#d00;stroke-width:8;stroke-linecap:round"/>
        <circle cx="0" cy="-108" r="8" fill="none" style="stroke:#d00;stroke-width:7"/>
      </g>
      <circle cx="250" cy="250" r="10" stroke="#d00" fill="#d00"/>
    </svg>
    </div>
    <div class="control">
      <section>
        <button v-for="val in ['hourAdd', 'minuteAdd', 'secondAdd']" :key="val" @click="btnChange(val)">+</button>
      </section>
      <section class="time">
        <span>{{ hour > 9 ? Math.floor(hour) : '0' + Math.floor(hour) }}</span>
        <span>:</span>
        <span>{{ minute > 9 ? Math.floor(minute) : '0' + Math.floor(minute) }}</span>
        <span>:</span>
        <span>{{ second > 9 ? Math.floor(second) : '0' + Math.floor(second) }}</span>
      </section>
      <section>
        <button v-for="val in ['hourAdd', 'minuteAdd', 'secondAdd']" :key="val" @click="btnChange(val, false)">-</button>
      </section>
    </div>
  </div>
</template>
<script>
export default {
  mounted () {
    let now = new Date()
    this.time.hour = now.getHours()
    this.time.minute = now.getMinutes()
    this.init()
    window.onresize = () => {
      this.vw = document.body.clientWidth
    }
  },
  data () {
    return {
      interval: null,
      vw: document.body.clientWidth,
      time: {
        hour: 0,
        minute: 0,
        second: 0,
        hourAdd: 0,
        minuteAdd: 0,
        secondAdd: 0
      }
    }
  },
  methods: {
    init () {
      this.time.second = new Date().getSeconds()
      this.interval = setInterval(() => {
        this.time.second = new Date().getSeconds()
      }, 1000)
    },
    btnChange (type, flag = true) {
      clearInterval(this.interval)
      if (flag) {
        this.time[type] += 1
      } else {
        this.time[type] -= 1
      }
      this.init()
    },
    dragChange (prop) {
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
        if (prop === 'hour') {
          self.time.hourAdd += Math.floor(rotate / 360 * 12) - self.hour
          self.time.minuteAdd += rotate % 30 / 30 * 60 - self.minute
        } else if (prop === 'minute') {
          self.time.minuteAdd += Math.floor(rotate / 360 * 60) - self.minute
        } else if (prop === 'second') {
          self.time.secondAdd += rotate / 360 * 60 - self.second
        }
      }
      function up () {
        self.dragAddTime = 0
        document.removeEventListener('mousemove', move)
        document.removeEventListener('mouseup', up)
        self.init()
      }
      document.addEventListener('mouseup', up, false)
    }
  },
  watch: {
    time: {
      handler (time, oldTime) {
        if (Math.abs(time.hourAdd) === 24) {
          time.hourAdd = 0
        } else if (Math.abs(time.minuteAdd) === 60) {
          time.minuteAdd = 0
        } else if (Math.abs(time.secondAdd) === 60) {
          time.secondAdd = 0
        }
      },
      deep: true
    },
    second (val, oldVal) {
      if (val === 0 && oldVal === 59) {
        this.time.minute++
      } else if (val === 59 && oldVal === 0) {
        this.time.minute--
      }
    },
    minute (val, oldVal) {
      if (val === 0 && oldVal === 59) {
        this.time.hour++
      } else if (val === 59 && oldVal === 0) {
        this.time.hour--
      }
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
    hour () {
      let hour = this.time.hour + this.time.hourAdd
      if (hour > 23) {
        return hour - 24
      } else if (hour < 0) {
        return hour + 24
      } else {
        return hour
      }
    },
    minute () {
      let minute = this.time.minute + this.time.minuteAdd
      if (minute >= 60) {
        return minute - 60
      } else if (minute < 0) {
        return minute + 60
      } else {
        return minute
      }
    },
    second () {
      let second = this.time.second + this.time.secondAdd
      if (second >= 60) {
        return second - 60
      } else if (second < 0) {
        return second + 60
      } else {
        return second
      }
    }
  }
}
</script>
<style lang="scss">
* {
  padding: 0;
  margin: 0;
  border: 0;
}
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}
@mixin flex-column-center{
  display: flex;
  align-items: center;
  flex-direction: column;
}
:root {
  font-size: 1vw;
}
html{
  height: 100%;
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
    @include flex-column-center;
    width: 100%;
    height: 100%;
    #clock{
      width: 500px;
      height: 500px;
    }
    .control{
      width: 30rem;
      height: 10rem;
      @include flex-column-center;
      button{
        vertical-align: middle;
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
        span{
          display: inline-block;
          text-align: center;
          &:nth-of-type(2n+1) {
            width: 7rem;
          }
          &:nth-of-type(2n) {
            width: 0.3rem;
          }
        }
      }
    }
  }
}
</style>
