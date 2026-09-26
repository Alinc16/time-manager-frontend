<template>
  <div class="card">
    <div class="header-row">
      <h2>📅 Working Times</h2>
      <button v-if="userId" @click="openCreateModal" class="success">+ Add Working Time</button>
    </div>

    <div v-if="!userId" class="warning">
      Please select a user to view working times.
    </div>

    <div v-else>
      <!-- Date Filters -->
      <div class="filter-bar">
        <div class="filter-item">
          <label>Start Date:</label>
          <input v-model="filterStart" type="text" placeholder="YYYY-MM-DD 00:00:00" />
        </div>
        <div class="filter-item">
          <label>End Date:</label>
          <input v-model="filterEnd" type="text" placeholder="YYYY-MM-DD 23:59:59" />
        </div>
        <button @click="getWorkingTimes" class="primary">Filter</button>
        <button @click="clearFilters" class="secondary">Clear</button>
      </div>

      <!-- Table -->
      <div v-if="workingTimesList.length === 0" class="empty-state">
        No working times found for this user.
      </div>
      <table v-else class="data-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Start Time</th>
            <th>End Time</th>
            <th>Duration</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="wt in workingTimesList" :key="wt.id">
            <td>#{{ wt.id }}</td>
            <td>{{ wt.start }}</td>
            <td>{{ wt.end }}</td>
            <td>{{ calculateDuration(wt.start, wt.end) }}</td>
            <td>
              <button @click="openEditModal(wt)" class="small-btn edit">Edit</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <WorkingTime
      v-if="showModal"
      :user-id="userId"
      :item="selectedItem"
      @saved="onRecordSaved"
      @close="showModal = false"
    />
  </div>
</template>

<script>
import api from '../services/api';
import WorkingTime from './WorkingTime.vue';

export default {
  name: 'WorkingTimes',
  components: {
    WorkingTime
  },
  props: {
    userId: {
      type: [Number, String],
      default: null
    }
  },
  emits: ['times-updated'],
  data() {
    return {
      workingTimesList: [],
      filterStart: '',
      filterEnd: '',
      showModal: false,
      selectedItem: null
    };
  },
  watch: {
    userId(newVal) {
      if (newVal) {
        this.getWorkingTimes();
      } else {
        this.workingTimesList = [];
      }
    }
  },
  methods: {
    async getWorkingTimes() {
      if (!this.userId) return;
      try {
        let url = `/workingtimes/${this.userId}`;
        const params = [];
        if (this.filterStart) params.push(`start=${encodeURIComponent(this.filterStart)}`);
        if (this.filterEnd) params.push(`end=${encodeURIComponent(this.filterEnd)}`);
        if (params.length > 0) url += `?${params.join('&')}`;

        const res = await api.get(url);
        this.workingTimesList = res.data.data || res.data || [];
        this.$emit('times-updated', this.workingTimesList);
      } catch (err) {
        console.error('Failed to fetch working times:', err);
        this.workingTimesList = [];
      }
    },
    clearFilters() {
      this.filterStart = '';
      this.filterEnd = '';
      this.getWorkingTimes();
    },
    calculateDuration(start, end) {
      if (!start || !end) return '-';
      const s = new Date(start.replace(' ', 'T'));
      const e = new Date(end.replace(' ', 'T'));
      const diffMs = e - s;
      if (isNaN(diffMs) || diffMs < 0) return '-';
      const hours = Math.floor(diffMs / (1000 * 60 * 60));
      const mins = Math.floor((diffMs % (1000 * 60 * 60)) / (1000 * 60));
      return `${hours}h ${mins}m`;
    },
    openCreateModal() {
      this.selectedItem = null;
      this.showModal = true;
    },
    openEditModal(item) {
      this.selectedItem = { ...item };
      this.showModal = true;
    },
    onRecordSaved() {
      this.showModal = false;
      this.getWorkingTimes();
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
.header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}
.warning {
  padding: 12px;
  background-color: #fef3c7;
  color: #92400e;
  border-radius: 6px;
}
.filter-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  align-items: flex-end;
  margin-bottom: 16px;
}
.filter-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 13px;
}
input {
  padding: 6px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.data-table {
  width: 100%;
  border-collapse: collapse;
}
.data-table th, .data-table td {
  padding: 10px 12px;
  border-bottom: 1px solid #e5e7eb;
  text-align: left;
  font-size: 14px;
}
.data-table th {
  background-color: #f9fafb;
  font-weight: 600;
}
.empty-state {
  text-align: center;
  padding: 20px;
  color: #6b7280;
}
button {
  padding: 8px 14px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
}
button.primary { background-color: #3b82f6; color: white; }
button.secondary { background-color: #6b7280; color: white; }
button.success { background-color: #10b981; color: white; }
.small-btn {
  padding: 4px 8px;
  font-size: 12px;
}
.small-btn.edit { background-color: #f59e0b; color: white; }
</style>