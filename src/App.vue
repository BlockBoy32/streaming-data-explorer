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
        <h2>Streams Over Time</h2>
        <canvas id="streamsOverTimeChart"></canvas>
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
      const streamsOverTimeData = processStreamsOverTimeData(data);

      createStreamsChart(streamsChartData);
      createTopArtistsChart(topArtistsChartData);
      createArtistPieChart(artistPieChartData);
      createTopTracksChart(topTracksChartData);
      createStreamsOverTimeChart(streamsOverTimeData);
    };

    const loadStreamingData = () => {
      const requireContext = require.context('./assets/anthony-data', false, /StreamingHistory_music_.*\.json$/);
      const allData = requireContext.keys().map(requireContext); 
      return allData.flat(); 
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

    const processStreamsOverTimeData = (data) => {
      const dailyCounts = {};
      data.forEach(item => {
        const date = item.endTime.split(' ')[0]; 
        dailyCounts[date] = (dailyCounts[date] || 0) + item.msPlayed;
      });
      return Object.entries(dailyCounts).map(([date, count]) => ({ date, count }));
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

    const createStreamsOverTimeChart = (data) => {
      const ctx = document.getElementById('streamsOverTimeChart').getContext('2d');
      const labels = data.map(item => item.date);
      const counts = data.map(item => item.count);

      new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [{
            label: 'Total Streams Over Time (ms)',
            data: counts,
            borderColor: 'blue',
            fill: false,
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
``
