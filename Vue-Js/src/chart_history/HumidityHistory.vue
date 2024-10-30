<template>
  <div>
    <input type="date" id="datePicker" v-model="selectedDate" @change="fetchDataForDate" />
    <div>
      <canvas id="HumidityHistory" ref="chartRef" width="400" height="100"></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import axios from 'axios';
import { ref, onMounted } from 'vue';
import { Chart, registerables } from 'chart.js';
import 'chartjs-adapter-date-fns'; // Import the date-fns adapter
import io from 'socket.io-client';
// @ts-ignore
import annotationPlugin from 'chartjs-plugin-annotation';

Chart.register(...registerables, annotationPlugin);

export default {
  name: 'HumidityHistory',
  setup() {
    const chartRef = ref<HTMLCanvasElement | null>(null);
    const socket = io('http://localhost:3000');
    let humidityChart: Chart | undefined;
    const selectedDate = ref<string | null>(null);

    const createChart = (data: any) => {
      if (humidityChart) {
        humidityChart.destroy();
      }

      if (chartRef.value) {
        const ctx = chartRef.value.getContext('2d');
        if (ctx) {
          humidityChart = new Chart(ctx, {
            type: 'line',
            data: {
              labels: data.timestamps,
              datasets: [
                {
                  label: 'Sensor 1',
                  backgroundColor: '#009FB7',
                  borderColor: '#009FB7',
                  data: data.sensor1_humidity,
                },
                {
                  label: 'Sensor 2',
                  backgroundColor: '#04F06A',
                  borderColor: '#04F06A',
                  data: data.sensor2_humidity,
                },
                {
                  label: 'Sensor 3',
                  backgroundColor: '#FED766',
                  borderColor: '#FED766',
                  data: data.sensor3_humidity,
                },
                {
                  label: 'Sensor 4',
                  backgroundColor: '#FF0063',
                  borderColor: '#FF0063',
                  data: data.sensor4_humidity,
                },
                {
                  label: 'Sensor 5',
                  backgroundColor: '#744FC6',
                  borderColor: '#744FC6',
                  data: data.sensor5_humidity,
                },
                {
                  label: 'Sensor 6',
                  backgroundColor: '#17301C',
                  borderColor: '17301C',
                  data: data.sensor6_humidity,
                },
              ],
            },
            options: {
              responsive: true,
              aspectRatio: 3,
              scales: {
                x: {
                  type: 'time',
                  time: {
                    unit: 'minute',
                    tooltipFormat: 'yyyy-MM-dd HH:mm',
                    displayFormats: {
                      minute: 'HH:mm',
                    },
                  },
                  ticks: {
                    stepSize: 10,
                  },
                  title: {
                    display: true,
                    text: 'Waktu',
                  },
                },
                y: {
                  beginAtZero: true,
                  title: {
                    display: true,
                    text: 'Kelembapan (%RH)',
                  },
                },
              },
            },
          });
        }
      }
    };

    const fetchDataForDate = async () => {
      if (selectedDate.value) {
        try {
          const response = await axios.get('http://localhost:3000/api/copy-data');
          const data = response.data;

          const startOfDay = new Date(selectedDate.value);
          startOfDay.setHours(0, 0, 0, 0);
          const endOfDay = new Date(startOfDay);
          endOfDay.setHours(23, 59, 59, 999);

          console.log('Start of Day:', startOfDay);
          console.log('End of Day:', endOfDay);

          // Filter data
          const filteredDataMap = new Map();
          data.forEach((row: any) => {
            const rowDate = new Date(row.timestamp);
            if (rowDate >= startOfDay && rowDate <= endOfDay) {
              filteredDataMap.set(row.timestamp, row);
            }
          });

          const filteredData = Array.from(filteredDataMap.values());

          // Mengurutkan data yang telah difilter berdasarkan timestamp
          filteredData.sort((a, b) => new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime());

          const chartData = {
            timestamps: filteredData.map((row: any) => row.timestamp),
            sensor1_humidity: filteredData.map((row: any) => row.sensor1_humidity),
            sensor2_humidity: filteredData.map((row: any) => row.sensor2_humidity),
            sensor3_humidity: filteredData.map((row: any) => row.sensor3_humidity),
            sensor4_humidity: filteredData.map((row: any) => row.sensor4_humidity),
            sensor5_humidity: filteredData.map((row: any) => row.sensor5_humidity),
            sensor6_humidity: filteredData.map((row: any) => row.sensor6_humidity),
          }

          console.log('Filtered and Deduplicated Data:', chartData);

          createChart(chartData);
        } catch (error) {
          console.error('Error fetching data for date:', error);
          if (humidityChart) {
            humidityChart.destroy();
            humidityChart = undefined;
          }
        }
      }
    };

    onMounted(() => {
      // Mengatur tanggal pada saat ini
      const today = new Date().toISOString().substr(0, 10);
      selectedDate.value = today;

      // Fetch data
      fetchDataForDate();
    });

    return {
      chartRef,
      selectedDate,
      fetchDataForDate,
    };
  },
};
</script>