<template>
  <div class="__site" scoped>
    <div id="popup">
      <div class="popup__content">
        <h2>Zamierzasz zaktualizować konfigurację czujnika.</h2>
        <h3>Nowe ustawienia:</h3>
        <ul>
          <li>
            Częstotliwość odczytu:
            <span class="new"> co {{ this.refresh_time }} sek.</span>
          </li>
          <li>
            Minimalna wilgotność: <span class="new">{{ this.minMoist }}%</span>
          </li>
          <li>
            Maksymalna wilgotność: <span class="new">{{ this.maxMoist }}%</span>
          </li>
        </ul>
        <p>
          Zmiany zostaną wprowadzone po kolejnym odczycie lub restarcie
          urządzenia
        </p>
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
      <h1>Roślina {{ $route.params.plant }}</h1>
      <h4 id="error"></h4>
      <h4 id="info"></h4>
      <div class="amount-group">
        <h4 v-if="this.chartData" style="margin: 0">Ostatnie</h4>
        <input type="number" name="" id="" v-model="amount" />
        <h4 style="margin: 0">pomiarów</h4>
      </div>
      <canvas id="myChart"></canvas>
      <div v-if="this.chartData">
        <!-- <button @click="updateChart()">Aktualizuj dane</button> -->
        <div class="input-group">
          <label>Ilość pomiarów:</label>
          <input type="number" v-model="amount" />
          <hr />

          <label>Częstotliwość odświeżania (s):</label>
          <input type="number" v-model="refresh_time" />
          <label>Minimalna wilgotność:</label>
          <input type="number" v-model="minMoist" />
          <label>Maksymalna wilgotność:</label>
          <input type="number" v-model="maxMoist" />
          <button @click="showPopup()" style="margin-bottom: 80px">
            Zmień ustawienia
          </button>
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
      minMoist: null,
      refresh_time: null,
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
              borderColor: 'rgba(255, 99, 132, 1)',
              backgroundColor: 'rgba(255, 99, 132, 0.2)',
              hidden: false,
              pointStyle: 'circle',
              borderWidth: 5,
              pointRadius: 5,
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
              hidden: false,
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

    updateConfig() {
      axios
        .put(`http://localhost:5000/api/esp_1/config`, {
          maxMoist: this.maxMoist,
          minMoist: this.minMoist,
          refresh_time: this.refresh_time,
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
      axios
        .get(
          `http://localhost:5000/api/plant${this.$route.params.plant}/readings?amount=${this.amount}`
        )
        .then((res) => {
          this.refresh_time = res.data[res.data.length - 1].refresh_time
          this.maxMoist = res.data[res.data.length - 1].max_moist
          this.minMoist = res.data[res.data.length - 1].min_moist
          this.chartData = res.data
          this.makeGraph()
        })
        .catch((err) => {
          document.getElementById('error').innerHTML =
            'Brak połączenia z serwerem. Odśwież stronę by spróbować ponownie.'
          clearInterval(this.interval)
        })
    },
    updateChart() {
      axios
        .get(
          `http://localhost:5000/api/plant${this.$route.params.plant}/readings?amount=${this.amount}`
        )
        .then((res) => {
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
    }, 2000)
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
  width: 1000px;
  @media screen and (max-width: 768px) {
  }
}

#myChart {
  max-width: 100%;
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

#popup {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
  visibility: hidden;
  opacity: 0;
  transition: all 0.3s ease-in-out;
  &.active {
    visibility: visible;
    opacity: 1;
  }
  .popup__content {
    width: 800px;
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

#info {
  color: green;
  margin: 0;
  font-size: 20px;
  text-align: center;
}
</style>
