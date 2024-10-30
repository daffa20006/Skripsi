<template>
  <main id="Home-page">
    <h1>HISTORY DATA</h1>
    <div class="charts">
      <div class="chart-1">
        <h3>History Monitoring Temperatur</h3>
        <temperatureHistory/>
      </div>
    </div>
    <div class="charts">
      <div class="chart-1">
        <h3>History Monitoring Kelembapan</h3>
        <HumidityHistory/>
      </div>
    </div>
  </main>
</template>

<script>
import HumidityHistory from '../chart_history/HumidityHistory.vue';
import TemperatureHistory from '../chart_history/TemperatureHistory.vue';
import axios from 'axios';

export default {
  name: 'Home',
  components: {TemperatureHistory, HumidityHistory},
  methods: {
    DeleteData() {
      axios.post('http://localhost:3000/delete-data')
      .then(response => {
        if (response.data.success) {
          alert('Data telah dihapus');
          window.location.reload();
        } else {
          alert('Data gagal dihapus')
        }
        })
        .catch(error => {
          console.error('Error:', error);
          alert('Ada Error');
        });
      }
    }
  }
</script>



<style>
#about,
    Temperature, humidity {
  display: flex;
  flex-wrap: wrap;
}
.chart-container {
  width: 50%;
  height: 400px;
  position: relative
}

.charts{
  display: grid;
  grid-template-columns: 2fr 1fr;
  grid-gap: 10px;
  width: 145%;
  padding: 15px;
  padding-top: 10;
  padding-left: 0;
}

.chart-1{
  background: #fff;
  padding: 10px;
  border-radius: 5px;
  box-shadow: - 10px 25px rgba(0, 0, 0, 0.08);
}

.tombol_3{
	background: #ffffff;
	padding: 10px;
	border-radius: 0px;
	box-shadow: - 10px 10px rgba(255, 0, 0, 0.08);
	width: 8%;
	outline: 1px solid rgb(13, 6, 52);
  }
</style>
