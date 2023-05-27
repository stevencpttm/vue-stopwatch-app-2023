<template>
  <div id="app">
    <div id="stopwatch">{{ displayTime }}</div>
    <div id="stopwatch-controls">
      <button id="reset-and-lap" @click="resetAndLapHandler">
        {{ isStarted && !isStopped ? "Lap" : "Reset" }}
      </button>
      <button
        id="start-and-stop"
        @click="startAndStopHandler"
        :class="{ red: isStarted && !isStopped }"
      >
        {{ isStarted && !isStopped ? "Stop" : "Start" }}
      </button>
      <span id="brand">Stopwatch</span>
    </div>
    <ul id="stopwatch-records">
      <li
        :key="index"
        v-for="(lap, index) in lapsRecords"
        :class="{
          red: lap.isSlowest,
          green: lap.isFastest,
        }"
      >
        <span>Lap {{ lapsRecords.length - index }}</span>
        <span>{{ lap.display }}</span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      timer: null,
      displayTime: "00:00.00",
      startTime: null,
      currentTime: null,
      stopTime: null,
      stoppedTimeOffset: 0,
      stoppedTimeOffsetForLap: 0,
      isStarted: false,
      isStopped: false,
      lastLapTime: null,
      laps: [],
    };
  },
  methods: {
    startAndStopHandler() {
      if (!this.isStarted) {
        // Start - the beginning
        this.startTime = Date.now();
        this.lastLapTime = Date.now();

        this.timer = setInterval(() => {
          this.currentTime = Date.now();
          let elapsedTime = this.currentTime - this.startTime;
          this.displayTime = this.formatTime((elapsedTime / 1000).toFixed(2));
        }, 10);

        this.laps = [
          {
            time: 0,
            display: this.displayTime,
            isFastest: false,
            isSlowest: false,
          },
        ];

        this.isStarted = true;
        this.isStopped = false;
      } else if (this.isStarted && !this.isStopped) {
        // Started - and pressed Stop
        clearInterval(this.timer);
        this.stopTime = Date.now();
        this.isStopped = true;
      } else if (this.isStarted && this.isStopped) {
        // Stopped - Want to start again
        this.stoppedTimeOffset += Date.now() - this.stopTime;
        this.stoppedTimeOffsetForLap += Date.now() - this.stopTime;

        this.timer = setInterval(() => {
          this.currentTime = Date.now();
          let elapsedTime =
            this.currentTime - this.startTime - this.stoppedTimeOffset;
          this.displayTime = this.formatTime((elapsedTime / 1000).toFixed(2));
        }, 10);

        this.isStopped = false;
      }
    },
    resetAndLapHandler() {
      if (this.isStarted && this.isStopped) {
        // Reset button clicked
        this.timer = null;
        this.displayTime = "00:00.00";
        this.startTime = null;
        this.currentTime = null;
        this.stopTime = null;
        this.stoppedTimeOffset = 0;
        this.stoppedTimeOffsetForLap = 0;
        this.isStarted = false;
        this.isStopped = false;
        this.lastLapTime = null;
        this.laps = [];
      } else if (this.isStarted && !this.isStopped) {
        // Lap button clicked
        let elapsedTime =
          this.currentTime - this.lastLapTime - this.stoppedTimeOffsetForLap;
        let lapTime = (elapsedTime / 1000).toFixed(2);

        this.laps.push({
          time: parseFloat(lapTime),
          display: this.formatTime(lapTime),
        });

        this.lastLapTime = this.currentTime;
        this.stoppedTimeOffsetForLap = 0;
      }
    },
    formatTime(seconds) {
      let date = new Date(null);
      date.setSeconds(seconds);
      let result = date.toISOString().substr(14, 5);

      return `${result}.${(seconds + "").split(".")[1]}`;
    },
  },
  computed: {
    lapsRecords() {
      let lapsClone = [...this.laps];

      if (lapsClone.length > 2) {
        lapsClone = lapsClone.map((oneLap) => {
          return {
            ...oneLap,
            isFastest: false,
            isSlowest: false,
          };
        });

        let fastestIndex = 0;
        let slowestIndex = 0;

        for (let i = 0; i < lapsClone.length; i++) {
          if (lapsClone[i].time < lapsClone[fastestIndex].time) {
            fastestIndex = i;
          }
          if (lapsClone[i].time > lapsClone[slowestIndex].time) {
            slowestIndex = i;
          }
        }

        lapsClone[fastestIndex].isFastest = true;
        lapsClone[slowestIndex].isSlowest = true;
      }

      return lapsClone.reverse();
    },
  },
  watch: {
    displayTime() {
      let lapsClone = [...this.laps];

      let elapsedTime =
        this.currentTime - this.lastLapTime - this.stoppedTimeOffsetForLap;
      let lapTime = (elapsedTime / 1000).toFixed(2);

      lapsClone[lapsClone.length - 1] = {
        time: parseFloat(lapTime),
        display: this.formatTime(lapTime),
        isFastest: false,
        isSlowest: false,
      };

      this.laps = lapsClone;
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #000;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  color: #fff;
}

#stopwatch {
  flex: 0px 1 1;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 5rem;
}

#stopwatch-records {
  flex: 0px 1 1;
}

#stopwatch-records {
  list-style: none;
  padding: 0;
  margin: 0 1.2rem;
  overflow: auto;
}

#stopwatch-records li {
  border-bottom: 1px solid #323234;
  display: flex;
  padding: 0.8rem 0;
}

#stopwatch-records li.green {
  color: #2ed158;
}

#stopwatch-records li.red {
  color: #ff453a;
}

#stopwatch-records li:first-child {
  border-top: 1px solid #323234;
}

#stopwatch-records span {
  display: block;
  flex: 0px 1 1;
}

#stopwatch-records span:last-child {
  text-align: right;
}

#stopwatch-controls {
  flex: 80px 0 0;
  padding: 0.2rem 1.2rem;
  overflow: hidden;
  position: relative;
  margin-bottom: 1.5rem;
}

#brand {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translateX(-50%) translateY(-50%);
}

button#reset-and-lap,
button#start-and-stop {
  width: 80px;
  height: 80px;
  outline: none;
  background-color: #333;
  border: 2px solid #000;
  border-radius: 100%;
  color: #fff;
  font-size: 1rem;
  box-shadow: 0px 0px 0px 2px #333;
  float: left;
}

button#start-and-stop {
  background-color: #082a12;
  box-shadow: 0px 0px 0px 2px #082a12;
  color: #2ed158;
  float: right;
}

button#start-and-stop.red {
  background-color: #320e0b;
  box-shadow: 0px 0px 0px 2px #320e0b;
  color: #ff453a;
}
</style>
