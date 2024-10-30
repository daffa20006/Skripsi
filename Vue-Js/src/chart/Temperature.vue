<template>
    <div>
      <canvas id="Temperature" ref="chartRef" width="400" height="100"></canvas>
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
    name: 'Temperature',
    setup() {
      const chartRef = ref<HTMLCanvasElement | null>(null);
      const socket = io('http://localhost:3000');
      let temperatureChart: Chart | undefined;
  
      const createChart = (data: any) => {
        console.log('Creating chart with data:', data);
        if (chartRef.value) {
          const ctx = chartRef.value.getContext('2d');
          if (ctx) {
            temperatureChart = new Chart(ctx, {
              type: 'line',
              data: {
                labels: data.timestamps,
                datasets: [
                  {
                    label: 'Sensor 1',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#009FB7';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#009FB7';
                  },
                    data: data.sensor1_temperature,
                  },
                  {
                    label: 'Sensor 2',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#04F06A';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#04F06A';
                  },
                    data: data.sensor2_temperature,
                  },
                  {
                    label: 'Sensor 3',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#FED766';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#FED766';
                  },
                    data: data.sensor3_temperature,
                  },
                  {
                    label: 'Sensor 4',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#FF0063';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#FF0063';
                  },
                    data: data.sensor4_temperature,
                  },
                  {
                    label: 'Sensor 5',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#744FC6';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#744FC6';
                  },
                    data: data.sensor5_temperature,
                  },
                  {
                    label: 'Sensor 6',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#17301C';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 60 ? '#DE1A1A' : '#17301C';
                  },
                    data: data.sensor6_temperature,
                  },
                ],
              },
              options: {
                responsive: true,
                aspectRatio: 3,
                devicePixelRatio: 2,
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
                      text: 'Temperature (°C)',
                    },
                  },
                },
                plugins: {
                  annotation: {
                    annotations: {
                      threshold: {
                        type: 'line',
                        yMin: 60,
                        yMax: 60,
                        borderColor: '#DE1A1A',
                        borderWidth: 2,
                        borderDash: [8, 8],
                        label: {
                          content: '60°C',
                          display: true,
                          position: 'end',
                        },
                      },
                    },
                  },
                },
              },
            });
          } else {
            console.error('Failed to get context for chart');
          }
        } else {
          console.error('chartRef.value is null');
        }
      };
  
      const updateChart = (newData: any) => {
        console.log('Updating chart with data:', newData);
          if (temperatureChart && temperatureChart.data) {
            // Menggabungkan data baru dengan data yang telah ada
            let combinedData = (temperatureChart.data.labels as string []).map((timestamp, index) =>({
            timestamp,
            sensor1_temperature: temperatureChart?.data.datasets[0].data[index] as number,
            sensor2_temperature: temperatureChart?.data.datasets[1].data[index] as number,
            sensor3_temperature: temperatureChart?.data.datasets[2].data[index] as number,
            sensor4_temperature: temperatureChart?.data.datasets[3].data[index] as number,
            sensor5_temperature: temperatureChart?.data.datasets[4].data[index] as number,
            sensor6_temperature: temperatureChart?.data.datasets[5].data[index] as number,
          }));

        const newCombinedData = newData.timestamps.map((timestamp: string, index: number) => ({
          timestamp,
          sensor1_temperature: newData.sensor1_temperature[index],
          sensor2_temperature: newData.sensor2_temperature[index],
          sensor3_temperature: newData.sensor3_temperature[index],
          sensor4_temperature: newData.sensor4_temperature[index],
          sensor5_temperature: newData.sensor5_temperature[index],
          sensor6_temperature: newData.sensor6_temperature[index],
        }));

        combinedData = combinedData.concat(newCombinedData);

        // Urutkan data berdasarkan timestamp dan hapus duplikasi
        combinedData = combinedData
          .sort((a, b) => new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime())
          .filter((item, index, self) =>
            index === self.findIndex((t) => t.timestamp === item.timestamp)
          );

          // Pisahkan kembali data ke dalam array yang berbeda
          const sortedLabels2 = combinedData.map(item => item.timestamp);
          const sortedSensor1Data = combinedData.map(item => item.sensor1_temperature);
          const sortedSensor2Data = combinedData.map(item => item.sensor2_temperature);
          const sortedSensor3Data = combinedData.map(item => item.sensor3_temperature);
          const sortedSensor4Data = combinedData.map(item => item.sensor4_temperature);  
          const sortedSensor5Data = combinedData.map(item => item.sensor5_temperature);
          const sortedSensor6Data = combinedData.map(item => item.sensor6_temperature);  

          // Update data chart
          temperatureChart.data.labels = sortedLabels2;
          temperatureChart.data.datasets[0].data = sortedSensor1Data;
          temperatureChart.data.datasets[1].data = sortedSensor2Data;
          temperatureChart.data.datasets[2].data = sortedSensor3Data;
          temperatureChart.data.datasets[3].data = sortedSensor4Data;
          temperatureChart.data.datasets[4].data = sortedSensor5Data;
          temperatureChart.data.datasets[5].data = sortedSensor6Data;

          temperatureChart.update();
        }
      };
  
      onMounted(async () => {
        try {
          const response = await axios.get('http://localhost:3000/api/sensor-data');
          const data = {
            timestamps: response.data.map((row: any) => row.timestamp),
            sensor1_temperature: response.data.map((row: any) => row.sensor1_temperature),
            sensor2_temperature: response.data.map((row: any) => row.sensor2_temperature),
            sensor3_temperature: response.data.map((row: any) => row.sensor3_temperature),
            sensor4_temperature: response.data.map((row: any) => row.sensor4_temperature),
            sensor5_temperature: response.data.map((row: any) => row.sensor5_temperature),
            sensor6_temperature: response.data.map((row: any) => row.sensor6_temperature),

          };
          console.log('Fetched data from API:', data);
          createChart(data);
  
          socket.on('sensorData', (data: any) => {
            const updatedData2 = {
              timestamps: Array.isArray(data) ? data.map((row: any) => row.timestamp) : [data.timestamp],
              sensor1_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor1_temperature) : [data.sensor1_temperature],
              sensor2_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor2_temperature) : [data.sensor2_temperature],
              sensor3_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor3_temperature) : [data.sensor3_temperature],
              sensor4_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor4_temperature) : [data.sensor4_temperature],
              sensor5_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor5_temperature) : [data.sensor5_temperature],
              sensor6_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor6_temperature) : [data.sensor6_temperature],
            };
            console.log('Received new data from socket:', updatedData2);
            updateChart(updatedData2);
          });
        } catch (error) {
          console.error('Error fetching data:', error);
        }
      });
  
      return {
        chartRef,
      };
    },
  };
  </script>
  
