<template>
  <div>
    <div class="time">
      <div class="time__content time__content--hour">
        <label>Hour:</label>
        <input v-model.number="time.hour" type="number" min="0" max="23">
      </div>
      <div class="time__content time__content--minute">
        <label>Minute:</label>
        <input v-model.number="time.minute" type="number" min="0" max="59">
      </div>
      <div class="time__content time__content--month">
        <label>Month:</label>
        <input v-model.number="time.month" type="number" min="1" max="12">
      </div>
      <div class="time__content time__content--weekday">
        <label>Weekday:</label>
        <select v-model.number="time.weekday">
          <option selected value="0">Monday</option>
          <option value="1">Tuesday</option>
          <option value="2">Wednesday</option>
          <option value="3">Thursday</option>
          <option value="4">Friday</option>
          <option value="5">Saturday</option>
          <option value="6">Sunday</option>
        </select>
      </div>
    </div>
    <div class="stations">
      <label>Station:</label>
      <select v-model="selected">
        <option v-for="(key, index) in keysSplice" v-bind:value="index" v-bind:key="key">
          {{ key | parseKey }}
        </option>
      </select>
    </div>
    <div class="prediction">{{prediction}}</div>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs';

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
    time: {
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
      if(this.time.month < 6 || this.time.month > 10) {
        this.prediction = "No data for these months 🙅‍♂️"
        return false
      }
      
      let predictor = new Array(this.keysLength).fill(0)
      let week = this.handleWeekday()
      predictor = [this.time.month, this.time.hour, this.time.minute, this.stationSize, ...week, ...predictor]
      
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
    handleWeekday: function () {
      let weekArray = new Array(7).fill(0)
      weekArray[this.time.weekday] = 1
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
      time : {
        month: 6,
        hour: 0,
        minute: 0,
        weekday: 0
      }
    }
  },
  filters: {
    parseKey: function (value) {
      let val = value.replace('X.name._', '')
      return val
    }
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
    font-weight: bold;
    margin-bottom: 1em;
    margin-top: 1em;
  }
</style>
