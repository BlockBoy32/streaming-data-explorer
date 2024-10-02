<template>
  <div id="app">
    <h1>Streaming Data Explorer</h1>
    <h2>Number of Streams by Month</h2>
    <canvas id="streamsChart" width="400" height="200"></canvas>
    
    <h2>Top 10 Artists by Total Streams</h2>
    <canvas id="topArtistsChart" width="400" height="200"></canvas>
  </div>
</template>

<script>
import { onMounted } from 'vue';
import { Chart, registerables } from 'chart.js'; // Import registerables

// Register all necessary components
Chart.register(...registerables);

export default {
  name: 'App',
  setup() {
    const fetchData = () => {
      const data = loadStreamingData(); // Load and format the streaming data
      const streamsChartData = processStreamsData(data);
      const topArtistsChartData = processTopArtistsData(data);
      createStreamsChart(streamsChartData);
      createTopArtistsChart(topArtistsChartData);
    };

    const loadStreamingData = () => {
      const requireContext = require.context('./assets/anthony-data', false, /StreamingHistory_music_.*\.json$/);
      const allData = requireContext.keys().map(requireContext); // Load all matching JSON files
      return allData.flat(); // Flatten the array to combine all data into a single array
    };

    const processStreamsData = (data) => {
      const monthlyCounts = {};
      data.forEach(item => {
        const month = item.endTime.split('-')[1]; // Extract month from endTime
        monthlyCounts[month] = (monthlyCounts[month] || 0) + item.msPlayed; // Sum msPlayed
      });
      return Object.entries(monthlyCounts).map(([month, count]) => ({ month, count }));
    };

    const processTopArtistsData = (data) => {
      const artistCounts = {};
      data.forEach(item => {
        artistCounts[item.artistName] = (artistCounts[item.artistName] || 0) + item.msPlayed; // Sum msPlayed
      });
      return Object.entries(artistCounts)
        .map(([artist, totalStreams]) => ({ artist, totalStreams }))
        .sort((a, b) => b.totalStreams - a.totalStreams) // Sort by totalStreams
        .slice(0, 10); // Get top 10 artists
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
            label: 'Number of Streams (ms)',
            data: counts,
            backgroundColor: 'skyblue',
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          }
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
            backgroundColor: 'lightgreen',
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          }
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
    margin: 20px;
}

h1 {
    color: #333;
}

h2 {
    margin-top: 40px;
}

canvas {
    max-width: 100%;
    height: auto;
}
</style>