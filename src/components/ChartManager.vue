<template>
  <div class="card">
    <h2>📊 Chart Manager</h2>

    <div v-if="!userId" class="warning">
      Please select a user to view charts.
    </div>

    <div v-else-if="workingTimes.length === 0" class="empty-state">
      No working time data available to display charts.
    </div>

    <div v-else class="charts-grid">
      <!-- 1. Bar Chart -->
      <div class="chart-container">
        <h3>📅 Daily Working Hours (Bar)</h3>
        <Bar :data="barChartData" :options="chartOptions" />
      </div>

      <!-- 2. Line Chart -->
      <div class="chart-container">
        <h3>📈 Working Trend Over Time (Line)</h3>
        <Line :data="lineChartData" :options="chartOptions" />
      </div>

      <!-- 3. Doughnut Chart -->
      <div class="chart-container">
        <h3>🎯 Weekly Target Balance (Doughnut)</h3>
        <div class="doughnut-wrapper">
          <Doughnut :data="doughnutChartData" :options="chartOptions" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  ArcElement
} from 'chart.js';
import { Bar, Line, Doughnut } from 'vue-chartjs';

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  ArcElement
);

export default {
  name: 'ChartManager',
  components: {
    Bar,
    Line,
    Doughnut
  },
  props: {
    userId: {
      type: [Number, String],
      default: null
    },
    workingTimes: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false
      }
    };
  },
  computed: {
    processedDailyData() {
      const dailyMap = {};

      this.workingTimes.forEach(item => {
        if (!item.start || !item.end) return;
        const startDate = item.start.split(' ')[0];
        const s = new Date(item.start.replace(' ', 'T'));
        const e = new Date(item.end.replace(' ', 'T'));
        const hours = (e - s) / (1000 * 60 * 60);

        if (hours > 0) {
          dailyMap[startDate] = (dailyMap[startDate] || 0) + Number(hours.toFixed(2));
        }
      });

      const labels = Object.keys(dailyMap).sort();
      const values = labels.map(day => dailyMap[day]);

      return { labels, values };
    },
    barChartData() {
      const { labels, values } = this.processedDailyData;
      return {
        labels: labels.length ? labels : ['No Data'],
        datasets: [
          {
            label: 'Hours Worked',
            backgroundColor: '#3b82f6',
            data: values.length ? values : [0]
          }
        ]
      };
    },
    lineChartData() {
      const { labels, values } = this.processedDailyData;
      return {
        labels: labels.length ? labels : ['No Data'],
        datasets: [
          {
            label: 'Work Trend (Hours)',
            borderColor: '#10b981',
            backgroundColor: 'rgba(16, 185, 129, 0.2)',
            tension: 0.3,
            fill: true,
            data: values.length ? values : [0]
          }
        ]
      };
    },
    doughnutChartData() {
      const totalHours = this.processedDailyData.values.reduce((sum, h) => sum + h, 0);
      const targetWeeklyHours = 35;
      const remainingTarget = Math.max(0, targetWeeklyHours - totalHours);

      return {
        labels: ['Logged Hours', 'Remaining Target'],
        datasets: [
          {
            backgroundColor: ['#6366f1', '#e5e7eb'],
            data: [Number(totalHours.toFixed(1)), Number(remainingTarget.toFixed(1))]
          }
        ]
      };
    }
  }
};
</script>

<style scoped>
.card {
  background: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  margin-bottom: 20px;
}
.warning {
  padding: 12px;
  background-color: #fef3c7;
  color: #92400e;
  border-radius: 6px;
}
.empty-state {
  text-align: center;
  padding: 24px;
  color: #6b7280;
}
.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 20px;
  margin-top: 15px;
}
.chart-container {
  background: #f9fafb;
  padding: 16px;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
  min-height: 280px;
  display: flex;
  flex-direction: column;
}
.chart-container h3 {
  font-size: 14px;
  margin-bottom: 12px;
  color: #374151;
}
.doughnut-wrapper {
  position: relative;
  height: 220px;
  display: flex;
  justify-content: center;
}
</style>