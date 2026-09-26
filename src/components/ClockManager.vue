<template>
  <div class="card">
    <h2>⏱️ Clock Manager</h2>

    <div v-if="!userId" class="warning">
      Please select a user above to manage clock-in / clock-out status.
    </div>

    <div v-else class="content">
      <div class="status-panel">
        <span class="badge" :class="clockin ? 'active' : 'inactive'">
          {{ clockin ? '🟢 Currently Clocked In' : '🔴 Currently Clocked Out' }}
        </span>
        <p v-if="startDateTime" class="time-info">
          Start Time: <strong>{{ startDateTime }}</strong>
        </p>
      </div>

      <div class="actions">
        <button 
          @click="clock" 
          :class="clockin ? 'danger' : 'success'"
          class="btn-large"
        >
          {{ clockin ? 'Clock Out' : 'Clock In' }}
        </button>

        <button @click="refresh" class="secondary">
          Refresh
        </button>
      </div>

      <p v-if="message" :class="['message', messageType]">{{ message }}</p>
    </div>
  </div>
</template>

<script>
import api from '../services/api';

export default {
  name: 'ClockManager',
  props: {
    userId: {
      type: [Number, String],
      default: null
    }
  },
  emits: ['clock-updated'],
  data() {
    return {
      startDateTime: null,
      clockin: false,
      message: '',
      messageType: 'info'
    };
  },
  watch: {
    userId(newVal) {
      if (newVal) {
        this.refresh();
      } else {
        this.startDateTime = null;
        this.clockin = false;
      }
    }
  },
  methods: {
    formatDate(dateObj) {
      const pad = (n) => String(n).padStart(2, '0');
      const year = dateObj.getFullYear();
      const month = pad(dateObj.getMonth() + 1);
      const day = pad(dateObj.getDate());
      const hours = pad(dateObj.getHours());
      const minutes = pad(dateObj.getMinutes());
      const seconds = pad(dateObj.getSeconds());
      return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
    },
    notify(text, type = 'info') {
      this.message = text;
      this.messageType = type;
      setTimeout(() => { this.message = ''; }, 3500);
    },
    async refresh() {
      if (!this.userId) return;
      try {
        const res = await api.get(`/clocks/${this.userId}`);
        const clockData = res.data.data || res.data;
        
        if (clockData && clockData.status !== undefined) {
          this.clockin = clockData.status;
          this.startDateTime = clockData.status ? (clockData.time || this.formatDate(new Date())) : null;
        } else {
          this.clockin = false;
          this.startDateTime = null;
        }
      } catch (err) {
        this.clockin = false;
        this.startDateTime = null;
      }
    },
    async clock() {
      if (!this.userId) return;

      const currentTimeFormatted = this.formatDate(new Date());
      const nextStatus = !this.clockin;

      const payload = {
        clock: {
          time: currentTimeFormatted,
          status: nextStatus
        }
      };

      try {
        const res = await api.post(`/clocks/${this.userId}`, payload);
        const result = res.data.data || res.data;

        this.clockin = result.status;
        this.startDateTime = this.clockin ? result.time : null;

        const statusMsg = this.clockin ? 'Clocked in successfully.' : 'Clocked out successfully.';
        this.notify(statusMsg, 'success');

        this.$emit('clock-updated');
      } catch (err) {
        console.error('Clock error:', err);
        this.notify('Failed to update clock status.', 'danger');
      }
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
  font-size: 14px;
}
.status-panel {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 16px;
}
.badge {
  display: inline-block;
  padding: 6px 12px;
  border-radius: 20px;
  font-weight: 600;
  width: fit-content;
  font-size: 14px;
}
.badge.active { background: #dcfce7; color: #166534; }
.badge.inactive { background: #fee2e2; color: #991b1b; }
.time-info {
  margin: 0;
  font-size: 14px;
  color: #374151;
}
.actions {
  display: flex;
  gap: 12px;
  align-items: center;
}
button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
}
.btn-large {
  padding: 10px 20px;
  font-size: 15px;
}
button.secondary { background-color: #6b7280; color: white; }
button.success { background-color: #10b981; color: white; }
button.danger { background-color: #ef4444; color: white; }
.message {
  margin-top: 12px;
  padding: 8px;
  border-radius: 4px;
  font-size: 14px;
}
.message.success { background: #d1fae5; color: #065f46; }
.message.danger { background: #fee2e2; color: #991b1b; }
.message.info { background: #e0f2fe; color: #075985; }
</style>