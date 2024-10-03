<template>
  <div id="app">
    <h1>Streaming Data Explorer</h1>
    <div class="chart-container">
      <div class="chart-wrapper">
        <h2>Number of Streams by Month</h2>
        <canvas id="streamsChart"></canvas>
      </div>
      
      <div class="chart-wrapper">
        <h2>Top 10 Artists by Total Streams</h2>
        <canvas id="topArtistsChart"></canvas>
      </div>

      <div class="chart-wrapper">
        <h2>Total Streams by Artist</h2>
        <canvas id="artistPieChart"></canvas>
      </div>

      <div class="chart-wrapper">
        <h2>Most Played Tracks</h2>
        <canvas id="topTracksChart"></canvas>
      </div>

      <div class="chart-wrapper">
        <h2>Streams Heatmap (Day of Week & Time of Day)</h2>
        <canvas id="streamsHeatmapChart"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted } from 'vue';
import { Chart, registerables } from 'chart.js';

Chart.register(...registerables);

export default {
  name: 'App',
  setup() {
    const fetchData = () => {
      const data = loadStreamingData();
      const streamsChartData = processStreamsData(data);
      const topArtistsChartData = processTopArtistsData(data);
      const artistPieChartData = processArtistPieData(data);
      const topTracksChartData = processTopTracksData(data);
      const heatmapData = processHeatmapData(data);

      createStreamsChart(streamsChartData);
      createTopArtistsChart(topArtistsChartData);
      createArtistPieChart(artistPieChartData);
      createTopTracksChart(topTracksChartData);
      createHeatmapChart(heatmapData);
    };

    const loadStreamingData = () => {
      const requireContext = require.context('./assets/anthony-data', false, /StreamingHistory_music_.*\.json$/);
      const allData = requireContext.keys().map(requireContext); 
      return allData.flat(); 
    };

    const processHeatmapData = (data) => {
      const heatmap = {};
      for (let i = 0; i < 7; i++) {
        heatmap[i] = {};
        for (let j = 0; j < 24; j++) {
          heatmap[i][j] = 0;
        }
      }

      data.forEach(item => {
        const dateTime = new Date(item.endTime);
        const dayOfWeek = dateTime.getDay(); // Sunday = 0, Monday = 1, ..., Saturday = 6
        const hourOfDay = dateTime.getHours(); // 0 to 23 hours
        heatmap[dayOfWeek][hourOfDay] += item.msPlayed;
      });

      const result = [];
      for (let day = 0; day < 7; day++) {
        for (let hour = 0; hour < 24; hour++) {
          result.push({
            x: hour,
            y: day,
            r: Math.sqrt(heatmap[day][hour]) / 1000, // Adjust the size based on the value
            value: heatmap[day][hour]
          });
        }
      }

      return result;
    };

    const createHeatmapChart = (data) => {
      const ctx = document.getElementById('streamsHeatmapChart').getContext('2d');

      new Chart(ctx, {
        type: 'bubble',
        data: {
          datasets: [{
            label: 'Heatmap of Streams by Time',
            data: data,
            backgroundColor: 'rgba(0, 150, 136, 0.6)', // Greenish color with transparency
            borderColor: 'rgba(0, 150, 136, 1)', // Solid green for border
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          scales: {
            x: {
              type: 'linear',
              position: 'bottom',
              ticks: {
                stepSize: 1,
                max: 23,
                callback: (value) => `${value}:00` // Show time of day in hours
              },
              title: {
                display: true,
                text: 'Hour of Day'
              }
            },
            y: {
              type: 'linear',
              ticks: {
                stepSize: 1,
                max: 6,
                callback: (value) => ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'][value]
              },
              title: {
                display: true,
                text: 'Day of Week'
              }
            }
          },
          plugins: {
            tooltip: {
              callbacks: {
                label: function(context) {
                  return `Streams: ${context.raw.value}`;
                }
              }
            }
          }
        }
      });
    };

    const processStreamsData = (data) => {
      const monthlyCounts = {};
      data.forEach(item => {
        const month = item.endTime.split('-')[1]; 
        monthlyCounts[month] = (monthlyCounts[month] || 0) + item.msPlayed;
      });
      return Object.entries(monthlyCounts).map(([month, count]) => ({ month, count }));
    };

    const processTopArtistsData = (data) => {
      const artistCounts = {};
      data.forEach(item => {
        artistCounts[item.artistName] = (artistCounts[item.artistName] || 0) + item.msPlayed;
      });
      return Object.entries(artistCounts)
        .map(([artist, totalStreams]) => ({ artist, totalStreams }))
        .sort((a, b) => b.totalStreams - a.totalStreams)
        .slice(0, 10);
    };

    const processArtistPieData = (data) => {
      const artistCounts = {};
      data.forEach(item => {
        artistCounts[item.artistName] = (artistCounts[item.artistName] || 0) + item.msPlayed;
      });

      const topArtists = Object.entries(artistCounts)
        .map(([artist, totalStreams]) => ({ artist, totalStreams }))
        .sort((a, b) => b.totalStreams - a.totalStreams)
        .slice(0, 10); 

      const otherTotal = Object.entries(artistCounts)
        .slice(10)
        .reduce((sum, [, totalStreams]) => sum + totalStreams, 0); 

      const topArtistsData = topArtists.map(({ artist, totalStreams }) => ({ artist, totalStreams }));
      if (otherTotal > 0) {
        topArtistsData.push({ artist: 'Other', totalStreams: otherTotal });
      }

      return topArtistsData;
    };

    const processTopTracksData = (data) => {
      const trackCounts = {};
      data.forEach(item => {
        trackCounts[item.trackName] = (trackCounts[item.trackName] || 0) + item.msPlayed;
      });
      return Object.entries(trackCounts)
        .map(([track, totalStreams]) => ({ track, totalStreams }))
        .sort((a, b) => b.totalStreams - a.totalStreams)
        .slice(0, 10);
    };

    const createArtistPieChart = (data) => {
      const ctx = document.getElementById('artistPieChart').getContext('2d');
      const labels = data.map(item => item.artist);
      const counts = data.map(item => item.totalStreams);

      new Chart(ctx, {
        type: 'pie',
        data: {
          labels: labels,
          datasets: [{
            data: counts,
            backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0', '#9966FF'],
          }]
        },
        options: {
          responsive: true,
        }
      });
    };

    const createTopTracksChart = (data) => {
      const ctx = document.getElementById('topTracksChart').getContext('2d');
      const labels = data.map(item => item.track);
      const counts = data.map(item => item.totalStreams);

      new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'Total Streams (ms)',
            data: counts,
            backgroundColor: 'lightcoral',
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          },
          responsive: true
        }
      });
    };

    const createStreamsChart = (data) => {
      const ctx = document.getElementById('streamsChart').getContext('2d');
      const labels = data.map(item => item.month);
      const counts = data.map(item => item.count);

      new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'Streams (ms)',
            data: counts,
            backgroundColor: 'rgba(75, 192, 192, 0.2)',
            borderColor: 'rgba(75, 192, 192, 1)',
            borderWidth: 1
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          },
          responsive: true
        }
      });
    };

    const createTopArtistsChart = (data) => {
      const ctx = document.getElementById('topArtistsChart').getContext('2d');
      const labels = data.map(item => item.artist);
      const counts = data.map(item => item.totalStreams);

      new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'Total Streams (ms)',
            data: counts,
            backgroundColor: 'lightblue',
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          },
          responsive: true
        }
      });
    };

    onMounted(fetchData);
  }
};
</script>

<style>
body {
    font-family: Arial, sans-serif;
    margin: 10px;
}

h1 {
    color: #333;
    text-align: center;
}

h2 {
    margin-top: 10px;
    text-align: center;
}

.chart-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
    justify-items: center;
}

.chart-wrapper {
    width: 100%;
    max-width: 500px;
}

canvas {
    width: 100% !important;
    height: auto !important;
}
</style>
