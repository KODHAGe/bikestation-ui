<template>
  <div>
    <div class="prediction-wrapper">
      <div class="prediction">{{prediction}}</div>
    </div>
    <div class="time">
      <datetime class="time" v-model="date" type="datetime"></datetime>
    </div>
    <div class="stations">
      <select v-model="selected">
        <option v-for="(key, index) in keysSplice" v-bind:value="index" v-bind:key="key">
          {{ key | parseKey }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'
import { Datetime } from 'vue-datetime'

// You need a specific loader for CSS files
import 'vue-datetime/dist/vue-datetime.css'


export default {
  name: 'PredictionControl',
  mounted: async function () {
    this.model = await tf.loadModel('https://s3.eu-north-1.amazonaws.com/bikestations-tfjs/bikes-latest/model.json');
    await this.handleKeys()
    this.predict()
  },
  watch: {
    selected: function() {
      this.predict()
    },
    date: {
      handler: function () {
        this.predict()
      },
      deep: true
    }
  },
  methods: {
    handleKeys: async function () {
      this.keys = await this.getKeys()
      this.keys = await this.keys.json()
      this.keysSplice = [...this.keys]
      this.keysRemoved = this.keysSplice.splice(0,11)
      this.keysLength = this.keysSplice.length
    },
    getKeys: async function () {
      try {
        let keys = fetch('https://s3.eu-north-1.amazonaws.com/bikestations-tfjs/keys.json')
        return keys
      } catch (error) {
        // eslint-disable-next-line
        console.log(error)
      }
    },
    predict: async function () {
      let predictor = new Array(this.keysLength).fill(0)
      let time = new Date(this.date)
      let week = this.handleWeekday(time)
      if(time.setHours(0,0,0,0) === new Date().setHours(0,0,0,0)) {
        this.prediction = "Choose a date & station!"
        return false
      }
      if(time.getMonth() < 6 || time.getMonth() > 10) {
        this.prediction = "No data for this month, try another 🙅‍♂️"
        return false
      }
      predictor = [time.getMonth(), time.getHours(), time.getMinutes(), this.stationSize, ...week, ...predictor]
      
      if(this.selected) {
        predictor[this.selected] = 1
      }
      
      let predict_this = tf.tensor2d(predictor, [1, predictor.length])
      let t0 = performance.now()
      let output = this.model.predict(predict_this).dataSync()
      let t1 = performance.now()
      // eslint-disable-next-line
      console.log("Prediction execution took " + (t1 - t0) + " milliseconds.")
      // eslint-disable-next-line
      console.log("The exact result being " + output)
      
      if(Math.floor(output) <= 0) {
        this.prediction = "Sorry, no bikes likely at that time 😓"
      } else {
        this.prediction = "Around " + Math.floor(output) + " bikes ought to be available 🚲"
      }
    },
    handleWeekday: function (time) {
      // This needs a check, current weekdays are weird
      let weekArray = new Array(7).fill(0)
      weekArray[time.getDay()] = 1
      return weekArray
    }
  },
  data: function() {
    return {
      model: Object,
      keys: Array,
      keysLength: Number,
      keysSplice: Array,
      keysRemoved: Array,
      prediction: 'w8, m8',
      selected: 228,
      stationSize: 20,
      date: new Date().toISOString()
    }
  },
  filters: {
    parseKey: function (value) {
      let val = value.replace('X.name._', '')
      return val
    }
  },
  components: {
    Datetime
  }
}
</script>

<style scoped>
  .time {
    display: flex;
    justify-content: center;
  }

  .time__content {
    margin: 0.5em;
    display: flex;
    flex-direction: column;
  }

  .time__content label {
    font-size: 0.8em;
    text-align: left;
  }
  
  .stations {
    margin-top: 1em;
  }

  .prediction {
    font-size: 3em;
    font-family: 'Gotham Rounded Bold';
    margin-bottom: 1em;
    margin-top: 1em;
    padding:20px;
  }

  .stations select {
    font-family: 'Gotham Rounded Medium';
    font-size: 1.5em;
    text-align: left;
    width: 294px;
    height: 48px;
    margin: auto;
    border-radius: 48px;
    -webkit-appearance: none;
    border: 2px solid rgba(0,0,0,1);
    background: #fcbc19;
    box-shadow: 0px 4px #b98d1c, 0px 6px rgba(0,0,0,1), 4px 12px rgba(0,0,0,0.2);
    padding-left: 1em;
    padding-right: 1em;
    transition: 200ms all;
    background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvLTg4NTktMSI/PjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIgImh0dHA6Ly93d3cudzMub3JnL0dyYXBoaWNzL1NWRy8xLjEvRFREL3N2ZzExLmR0ZCI+PHN2ZyB2ZXJzaW9uPSIxLjEiIGlkPSJDYXBhXzEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHg9IjBweCIgeT0iMHB4IiB3aWR0aD0iMjkyLjM2MnB4IiBoZWlnaHQ9IjI5Mi4zNjJweCIgdmlld0JveD0iMCAwIDI5Mi4zNjIgMjkyLjM2MiIgc3R5bGU9ImVuYWJsZS1iYWNrZ3JvdW5kOm5ldyAwIDAgMjkyLjM2MiAyOTIuMzYyOyIgeG1sOnNwYWNlPSJwcmVzZXJ2ZSI+PGc+PHBhdGggZD0iTTI4Ni45MzUsNjkuMzc3Yy0zLjYxNC0zLjYxNy03Ljg5OC01LjQyNC0xMi44NDgtNS40MjRIMTguMjc0Yy00Ljk1MiwwLTkuMjMzLDEuODA3LTEyLjg1LDUuNDI0QzEuODA3LDcyLjk5OCwwLDc3LjI3OSwwLDgyLjIyOGMwLDQuOTQ4LDEuODA3LDkuMjI5LDUuNDI0LDEyLjg0N2wxMjcuOTA3LDEyNy45MDdjMy42MjEsMy42MTcsNy45MDIsNS40MjgsMTIuODUsNS40MjhzOS4yMzMtMS44MTEsMTIuODQ3LTUuNDI4TDI4Ni45MzUsOTUuMDc0YzMuNjEzLTMuNjE3LDUuNDI3LTcuODk4LDUuNDI3LTEyLjg0N0MyOTIuMzYyLDc3LjI3OSwyOTAuNTQ4LDcyLjk5OCwyODYuOTM1LDY5LjM3N3oiLz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PC9zdmc+);
    background-position: 95% 50%;
    background-size: 12px;
    background-repeat: no-repeat;
  }

  .stations select:hover {
    transform: translateY(3px);
    cursor: pointer;
    box-shadow: 0px 2px #a37c18, 0px 4px black, 2px 7px rgba(0,0,0,0.2);
    transition: 200ms all;
  }

  .stations select:active {
    transform: translateY(6px);
    cursor: pointer;
    box-shadow: none;
    transition: 200ms all;
  }

  /* vue-datetime style overrides */

  .time >>> .vdatetime-input {
    padding: 0px;
    font-family: 'Gotham Rounded Medium';
    font-size: 1.5em;
    text-align: left;
    width: 294px;
    height: 48px;
    margin: auto;
    border-radius: 48px;
    border: 2px solid rgba(0,0,0,1);
    background: #fcbc19;
    box-shadow: 0px 4px #b98d1c, 0px 6px rgba(0,0,0,1), 4px 12px rgba(0,0,0,0.2);
    padding-left: 1em;
    transition: 200ms all;
    background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvLTg4NTktMSI/PjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIgImh0dHA6Ly93d3cudzMub3JnL0dyYXBoaWNzL1NWRy8xLjEvRFREL3N2ZzExLmR0ZCI+PHN2ZyB2ZXJzaW9uPSIxLjEiIGlkPSJDYXBhXzEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHg9IjBweCIgeT0iMHB4IiB3aWR0aD0iNDg4LjE1MnB4IiBoZWlnaHQ9IjQ4OC4xNTJweCIgdmlld0JveD0iMCAwIDQ4OC4xNTIgNDg4LjE1MiIgc3R5bGU9ImVuYWJsZS1iYWNrZ3JvdW5kOm5ldyAwIDAgNDg4LjE1MiA0ODguMTUyOyIgeG1sOnNwYWNlPSJwcmVzZXJ2ZSI+PGc+PGc+PHBhdGggZD0iTTE3Ny44NTQsMjY5LjMxMWMwLTYuMTE1LTQuOTYtMTEuMDY5LTExLjA4LTExLjA2OWgtMzguNjY1Yy02LjExMywwLTExLjA3NCw0Ljk1NC0xMS4wNzQsMTEuMDY5djM4LjY2YzAsNi4xMjMsNC45NjEsMTEuMDc5LDExLjA3NCwxMS4wNzloMzguNjY1YzYuMTIsMCwxMS4wOC00Ljk1NiwxMS4wOC0xMS4wNzlWMjY5LjMxMUwxNzcuODU0LDI2OS4zMTF6Ii8+PHBhdGggZD0iTTI3NC40ODMsMjY5LjMxMWMwLTYuMTE1LTQuOTYxLTExLjA2OS0xMS4wNjktMTEuMDY5aC0zOC42N2MtNi4xMTMsMC0xMS4wNzQsNC45NTQtMTEuMDc0LDExLjA2OXYzOC42NmMwLDYuMTIzLDQuOTYxLDExLjA3OSwxMS4wNzQsMTEuMDc5aDM4LjY3YzYuMTA4LDAsMTEuMDY5LTQuOTU2LDExLjA2OS0xMS4wNzlWMjY5LjMxMXoiLz48cGF0aCBkPSJNMzcxLjExNywyNjkuMzExYzAtNi4xMTUtNC45NjEtMTEuMDY5LTExLjA3NC0xMS4wNjloLTM4LjY2NWMtNi4xMiwwLTExLjA4LDQuOTU0LTExLjA4LDExLjA2OXYzOC42NmMwLDYuMTIzLDQuOTYsMTEuMDc5LDExLjA4LDExLjA3OWgzOC42NjVjNi4xMTMsMCwxMS4wNzQtNC45NTYsMTEuMDc0LTExLjA3OVYyNjkuMzExeiIvPjxwYXRoIGQ9Ik0xNzcuODU0LDM2NS45NWMwLTYuMTI1LTQuOTYtMTEuMDc1LTExLjA4LTExLjA3NWgtMzguNjY1Yy02LjExMywwLTExLjA3NCw0Ljk1LTExLjA3NCwxMS4wNzV2MzguNjUzYzAsNi4xMTksNC45NjEsMTEuMDc0LDExLjA3NCwxMS4wNzRoMzguNjY1YzYuMTIsMCwxMS4wOC00Ljk1NiwxMS4wOC0xMS4wNzRWMzY1Ljk1TDE3Ny44NTQsMzY1Ljk1eiIvPjxwYXRoIGQ9Ik0yNzQuNDgzLDM2NS45NWMwLTYuMTI1LTQuOTYxLTExLjA3NS0xMS4wNjktMTEuMDc1aC0zOC42N2MtNi4xMTMsMC0xMS4wNzQsNC45NS0xMS4wNzQsMTEuMDc1djM4LjY1M2MwLDYuMTE5LDQuOTYxLDExLjA3NCwxMS4wNzQsMTEuMDc0aDM4LjY3YzYuMTA4LDAsMTEuMDY5LTQuOTU2LDExLjA2OS0xMS4wNzRWMzY1Ljk1eiIvPjxwYXRoIGQ9Ik0zNzEuMTE3LDM2NS45NWMwLTYuMTI1LTQuOTYxLTExLjA3NS0xMS4wNjktMTEuMDc1aC0zOC42N2MtNi4xMiwwLTExLjA4LDQuOTUtMTEuMDgsMTEuMDc1djM4LjY1M2MwLDYuMTE5LDQuOTYsMTEuMDc0LDExLjA4LDExLjA3NGgzOC42N2M2LjEwOCwwLDExLjA2OS00Ljk1NiwxMS4wNjktMTEuMDc0VjM2NS45NUwzNzEuMTE3LDM2NS45NXoiLz48cGF0aCBkPSJNNDQwLjI1NCw1NC4zNTR2NTkuMDVjMCwyNi42OS0yMS42NTIsNDguMTk4LTQ4LjMzOCw0OC4xOThoLTMwLjQ5M2MtMjYuNjg4LDAtNDguNjI3LTIxLjUwOC00OC42MjctNDguMTk4VjU0LjE0MmgtMTM3LjQ0djU5LjI2MmMwLDI2LjY5LTIxLjkzOCw0OC4xOTgtNDguNjIyLDQ4LjE5OEg5Ni4yMzVjLTI2LjY4NSwwLTQ4LjMzNi0yMS41MDgtNDguMzM2LTQ4LjE5OHYtNTkuMDVDMjQuNTc2LDU1LjA1Nyw1LjQxMSw3NC4zNTYsNS40MTEsOTguMDc3djM0Ni4wNjFjMCwyNC4xNjcsMTkuNTg4LDQ0LjAxNSw0My43NTUsNDQuMDE1aDM4OS44MmMyNC4xMzEsMCw0My43NTUtMTkuODg5LDQzLjc1NS00NC4wMTVWOTguMDc3QzQ4Mi43NDEsNzQuMzU2LDQ2My41NzcsNTUuMDU3LDQ0MC4yNTQsNTQuMzU0eiBNNDI2LjA5MSw0MjIuNTg4YzAsMTAuNDQ0LTguNDY4LDE4LjkxNy0xOC45MTYsMTguOTE3SDgwLjE0NGMtMTAuNDQ4LDAtMTguOTE2LTguNDczLTE4LjkxNi0xOC45MTdWMjQzLjgzNWMwLTEwLjQ0OCw4LjQ2Ny0xOC45MjEsMTguOTE2LTE4LjkyMWgzMjcuMDNjMTAuNDQ4LDAsMTguOTE2LDguNDczLDE4LjkxNiwxOC45MjFMNDI2LjA5MSw0MjIuNTg4TDQyNi4wOTEsNDIyLjU4OHoiLz48cGF0aCBkPSJNOTYuMTI4LDEyOS45NDVoMzAuMTYyYzkuMTU1LDAsMTYuNTc4LTcuNDEyLDE2LjU3OC0xNi41NjdWMTYuNTczQzE0Mi44NjgsNy40MTcsMTM1LjQ0NSwwLDEyNi4yOSwwSDk2LjEyOEM4Ni45NzIsMCw3OS41NSw3LjQxNyw3OS41NSwxNi41NzN2OTYuODA1Qzc5LjU1LDEyMi41MzMsODYuOTcyLDEyOS45NDUsOTYuMTI4LDEyOS45NDV6Ii8+PHBhdGggZD0iTTM2MS4wMzUsMTI5Ljk0NWgzMC4xNjJjOS4xNDksMCwxNi41NzItNy40MTIsMTYuNTcyLTE2LjU2N1YxNi41NzNDNDA3Ljc3LDcuNDE3LDQwMC4zNDcsMCwzOTEuMTk3LDBoLTMwLjE2MmMtOS4xNTQsMC0xNi41NzcsNy40MTctMTYuNTc3LDE2LjU3M3Y5Ni44MDVDMzQ0LjQ1OCwxMjIuNTMzLDM1MS44ODEsMTI5Ljk0NSwzNjEuMDM1LDEyOS45NDV6Ii8+PC9nPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48Zz48L2c+PGc+PC9nPjxnPjwvZz48L3N2Zz4=);
    background-position: 95% 50%;
    background-size: 12px;
    background-repeat: no-repeat;
  }

  .time >>> .vdatetime-input:hover {
    transform: translateY(2px);
    cursor: pointer;
    box-shadow: 0px 2px #a37c18, 0px 4px black, 2px 7px rgba(0,0,0,0.2);
    transition: 200ms all;
  }

  .time >>> .vdatetime-input:active {
    transform: translateY(6px);
    cursor: pointer;
    box-shadow: none;
    transition: 200ms all;
  }

  .time >>> .vdatetime-popup__header {
    background-color: #fcbc19;
    color: black;
    font-family: 'Gotham Rounded Medium';
  }

  .time >>> .vdatetime-calendar__month__day--selected > span > span {
    background: #fcbc19;
    color: black;
  }

  .time >>> .vdatetime-popup__actions__button {
    color: black;
    font-family: 'Gotham Rounded Medium';
  }

  .time >>> .vdatetime-calendar {
    font-family: 'Gotham Rounded Medium';
  }

  .time >>> .vdatetime-calendar__month {
    font-family: 'Gotham Rounded Book';
  }

  .time >>> .vdatetime-time-picker__item--selected {
    color: black;
  }

  .time >>> .vdatetime-time-picker__item:hover {
    font-size: 20px;
    color: #fcbc19;
  }

  .time >>> .vdatetime-time-picker__item--selected:hover {
    font-size: 32px;
    color: #fcbc19;
  }

  .time >>> .vdatetime-time-picker__item {
    transition: 0ms;
  }

</style>
