<template>
  <div class="__site" scoped>
    <div id="popup">
      <div class="popup__content">
        <div v-if="this.chartData">
          <div class="input-group">
            <div class="input-group__item">
              <label>Częstotliwość odczytu (s):</label>
              <input type="number" v-model="refresh_time" />
            </div>
            <div class="input-group__item">
              <label>Minimalna wilgotność (%):</label>
              <input type="number" v-model="minMoist" />
            </div>
            <div class="input-group__item">
              <label>Maksymalna wilgotność (%):</label>
              <input type="number" v-model="maxMoist" />
            </div>
            <div class="input-group__item">
              <label>Wysokość zbiornika (cm):</label>
              <input type="number" v-model="tank_height" />
            </div>
            <div class="input-group__item">
              <label>Impuls podlewania (s):</label>
              <input type="number" v-model="pump_time" />
            </div>
            <span style="color: #888; font-style: italic; font-size: 12px"
              >~{{
                (110 * (this.pump_time / 3.6)).toFixed(2) > 1000
                  ? (110 * (this.pump_time / 3600)).toFixed(2) + 'l'
                  : (110 * (this.pump_time / 3.6)).toFixed(2) + 'ml'
              }}</span
            >

            <div class="input-group__item">
              <label>Tryb Aquaman:</label>
              <input type="checkbox" v-model="non_stop_pump" />
            </div>
          </div>
        </div>
        <div class="btn-group">
          <button
            class="btn btn--primary"
            @click="updateConfig()"
            style="background: #6ee698"
          >
            Zatwierdź
          </button>
          <button
            class="btn btn--secondary"
            @click="closePopup()"
            style="background: #eee"
          >
            Anuluj
          </button>
        </div>
      </div>
    </div>
    <NavBar />
    <div class="content">
      <!-- <h1 style="margin: 0">{{ $route.params.plant }}</h1> -->
      <div class="plant__header flex">
        <h1>Roślina {{ $route.params.plant }}</h1>
        <button @click="showPopup()" class="btn btn--primary">
          Zmień ustawienia
        </button>
      </div>
      <h4 id="error"></h4>
      <h4 id="info"></h4>
      <div class="amount-group">
        <h4 v-if="this.chartData" style="margin: 0">Ostatnie</h4>
        <input type="number" name="" id="" v-model="amount" />
        <h4 style="margin: 0">pomiarów</h4>
      </div>
      <div class="chart-container">
        <canvas id="myChart"></canvas>
        <div class="water-level">
          <h3 style="z-index: 2">{{ this.water_level }}%</h3>
          <div class="wrapper">
            <div class="water-tank">
              <div
                class="water-bar"
                :style="{ height: this.water_level + '%' }"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto'
import axios from 'axios'

export default {
  data() {
    return {
      chart: null,
      chartData: null,
      amount: 16,
      interval: null,
      maxMoist: null,
      water_level: null,
      minMoist: null,
      refresh_time: null,
      pump_time: null,
      tank_height: null,
      non_stop_pump: null,
      app_refresh_time: 5000,
      endpoint: 'https://esplant.onrender.com',
      // endpoint: 'http://localhost:5000',
    }
  },
  methods: {
    changePump() {
      this.chart.data.datasets[0].hidden = !this.chart.data.datasets[0].hidden
      this.chart.update()
    },
    makeGraph() {
      const ctx = document.getElementById('myChart')
      this.chart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: this.chartData.map(
            (d) => d.createdAt.split('T')[1].split('.')[0]
          ),
          datasets: [
            {
              label: 'Nawodnienie',
              data: this.chartData.map((d) => d.moisture_level),
              // green color
              borderColor: 'rgba(75, 192, 192, 1)',
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              hidden: false,
              pointStyle: 'circle',
              borderWidth: 3,
              pointRadius: 3,
              pointHoverRadius: 15,
              cubicInterpolationMode: 'monotone',
              tension: 0.2,
              yAxisID: 'y',
            },
            {
              label: 'Poziom wody',
              data: this.chartData.map((d) => d.water_level),
              borderColor: 'rgba(54, 162, 235, 1)',
              backgroundColor: 'rgba(54, 162, 235, 0.2)',
              hidden: true,
              pointStyle: 'circle',
              borderWidth: 5,
              pointRadius: 5,
              pointHoverRadius: 15,
              cubicInterpolationMode: 'monotone',
              tension: 0.2,
              yAxisID: 'y',
            },
            {
              label: 'Maksymalna wilgotność',
              data: this.chartData.map((d) => d.max_moist),
              borderColor: 'rgba(0, 255, 86, 1)',
              backgroundColor: 'rgba(255, 206, 86, 0.2)',
              // line thickness
              borderWidth: 1,
              hidden: false,
              pointStyle: 'dash',
              pointRadius: 0,
              pointHoverRadius: 0,
              cubicInterpolationMode: 'monotone',
              tension: 0.2,
              yAxisID: 'y',
            },
            {
              label: 'Minimalna wilgotność',
              data: this.chartData.map((d) => d.min_moist),
              borderColor: 'rgba(255, 0, 86, 1)',
              // line thickness
              borderWidth: 1,
              hidden: false,
              pointStyle: 'dash',
              pointRadius: 0,
              pointHoverRadius: 0,
              cubicInterpolationMode: 'monotone',
              tension: 0.2,
              yAxisID: 'y',
            },
          ],
        },
        options: {
          scales: {
            y: {
              min: 0,
              max: 100,
              type: 'linear',
              display: true,
              position: 'left',
              beginAtZero: true,
              grid: {
                drawOnChartArea: false,
              },
            },
            y1: {
              display: false,
              beginAtZero: true,
              grid: {
                drawOnChartArea: false,
              },
            },
          },
          interaction: {},
          elements: {
            point: {
              radius: 5,
            },
          },
        },
      })
    },

    getConfig() {
      axios
        .get(`${this.endpoint}/api/esp_1/config`)
        .then((res) => {
          this.maxMoist = res.data.maxMoist
          this.minMoist = res.data.minMoist
          this.refresh_time = res.data.refresh_time / 1000
          this.non_stop_pump = res.data.non_stop_pump
          this.tank_height = res.data.tank_height
          this.pump_time = res.data.pump_time / 1000
          console.log('PUMP: ' + this.pump_time)
        })
        .catch((err) => {
          console.log(err)
        })
    },

    updateConfig() {
      axios
        .put(`${this.endpoint}/api/esp_1/config`, {
          max_moist: this.maxMoist,
          min_moist: this.minMoist,
          refresh_time: this.refresh_time * 1000,
          non_stop_pump: this.non_stop_pump,
          tank_height: this.tank_height,
          pump_time: this.pump_time * 1000,
        })
        .then((res) => {
          console.log(res)
          document.getElementById('info').innerHTML = 'Zmieniono ustawienia'
          setTimeout(() => {
            document.getElementById('info').innerHTML = ''
          }, 3000)
        })
        .catch((err) => {
          console.log(err)
        })
      this.closePopup()
    },

    createChart() {
      console.log('XDDD')
      axios
        .get(
          `${this.endpoint}/api/plant${this.$route.params.plant}/readings?amount=${this.amount}`
        )
        .then((res) => {
          console.log(res.data)
          this.refresh_time = res.data[res.data.length - 1].refresh_time / 1000
          this.maxMoist = res.data[res.data.length - 1].max_moist
          this.minMoist = res.data[res.data.length - 1].min_moist
          this.non_stop_pump = res.data[res.data.length - 1].non_stop_pump
          this.water_level = res.data[res.data.length - 1].water_level
          this.tank_height = res.data[res.data.length - 1].tank_height
          this.pump_time = res.data[res.data.length - 1].pump_time / 1000
          this.chartData = res.data
          this.makeGraph()
        })
        .catch((err) => {
          document.getElementById('error').innerHTML =
            'Brak połączenia z serwerem. Odśwież stronę by spróbować ponownie.'
          clearInterval(this.interval)
          console.log(err)
        })
    },

    updateChart() {
      axios
        .get(
          `${this.endpoint}/api/plant${this.$route.params.plant}/readings?amount=${this.amount}`
        )
        .then((res) => {
          console.log(res.data)
          this.chartData = res.data
          this.chart.data.labels = this.chartData.map(
            (d) => d.createdAt.split('T')[1].split('.')[0]
          )
          this.chart.data.datasets[0].data = this.chartData.map(
            (d) => d.moisture_level
          )
          this.chart.data.datasets[1].data = this.chartData.map(
            (d) => d.water_level
          )
          this.chart.data.datasets[2].data = this.chartData.map(
            (d) => d.max_moist
          )
          this.chart.data.datasets[3].data = this.chartData.map(
            (d) => d.min_moist
          )
          this.water_level = res.data[res.data.length - 1].water_level

          this.chart.update()
        })
        .catch((err) => {
          document.getElementById('error').innerHTML =
            'Brak połączenia z serwerem. Odśwież stronę by spróbować ponownie.'
          clearInterval(this.interval)
        })
    },

    showPopup() {
      document.getElementById('popup').classList.add('active')
      window.scrollTo(0, 0)
    },

    closePopup() {
      document.getElementById('popup').classList.remove('active')
    },
  },
  mounted() {
    this.createChart()
    this.interval = setInterval(() => {
      this.updateChart()
    }, this.app_refresh_time)
  },
  destroyed() {
    clearInterval(this.interval)
  },
}
</script>

<style lang="scss" scoped>
html {
  font-family: 'Kanit', sans-serif !important;
}
.__site {
  margin: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
}
.content {
  padding: 80px 30px 30px 30px;
  display: flex;
  flex-direction: column;
  width: 100vw;
  max-height: 100vh;
  @media screen and (max-width: 768px) {
  }
}

.chart-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

#myChart {
  max-width: 90%;
  max-height: 100% !important;
  @media screen and (max-width: 1300px) {
    max-width: 100%;
  }
}

.water-level {
  height: 100%;
  width: 10%;
  padding: 20px;
  display: flex;
  position: relative;
  @media screen and (max-width: 1300px) {
    display: none;
  }
}
h3 {
  position: absolute;
  bottom: 50px;
  color: rgb(23, 72, 117);
  left: 50%;
  transform: translateX(-50%);
}
.wrapper {
  position: relative;
  height: 100%;
  width: 100%;
  overflow: hidden;
  padding: 3px;
  border-radius: 8px;
  border: 1px solid #a1c4fd;
  border-top: 0;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
  .water-tank {
    padding: 0px;
    position: relative;
    height: 100%;
    .water-bar {
      position: absolute;
      bottom: 0;
      background-image: linear-gradient(120deg, #a1c4fd 0%, #c2e9fb 100%);
      width: 100%;
      border-radius: 8px;
      transition: 0.5s ease all;
    }
  }
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-top: 20px;
}

#error {
  color: red;
  margin: 0;
  font-size: 20px;
  text-align: center;
}

.amount-group {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 10px;
  margin-top: 20px;

  input[type='number'] {
    width: 50px;
    border-radius: 5px;
    border: 1px solid #ccc;
    padding: 5px;
    font-weight: bold;
    font-size: 16px;
    background: none;
    font-family: 'Roboto', sans-serif;
  }
}

.plant__header {
  display: flex;
  gap: 30px;
  align-items: center;
  input {
    border-radius: 5px;
  }
}

#popup {
  position: absolute;
  top: 0;
  right: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: flex-end;
  align-items: flex-start;
  padding: 30px;
  z-index: 100;
  visibility: hidden;
  opacity: 0;
  transition: all 0.3s ease-in-out;
  &.active {
    visibility: visible;
    opacity: 1;
  }
  .popup__content {
    width: 400px;
    margin: 0 20px;
    background: #fff;
    border-radius: 10px;
    padding: 20px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    text-align: center;
    .close {
      position: absolute;
      top: 10px;
      right: 10px;
      font-size: 20px;
      cursor: pointer;
    }
    ul {
      list-style: none;
      padding: 0;
      margin: 0;
      li .new {
        color: green;
        font-weight: bold;
      }
    }
    .btn-group {
      display: flex;
      flex-direction: row;
      gap: 10px;
      margin-top: 20px;
      button {
        width: 100px;
        border-radius: 5px;
        border: 1px solid #ccc;
        padding: 5px;
        font-weight: bold;
        font-size: 16px;
        background: none;
        cursor: pointer;
        font-family: 'Kanit', sans-serif !important;
      }
    }
  }
}

.input-group__item {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  input[type='number'] {
    width: 100px;
    border-radius: 5px;
    border: 1px solid #ccc;
    padding: 5px;
    font-weight: bold;
    font-size: 16px;
    background: none;
  }
}

#info {
  color: green;
  margin: 0;
  font-size: 20px;
  text-align: center;
}
</style>
