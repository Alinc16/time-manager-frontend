<template>
  <div class="working-time-modal">
    <div class="modal-card">
      <h3>{{ isEditMode ? 'Edit Working Time' : 'Add New Working Time' }}</h3>

      <form @submit.prevent="handleSubmit" class="form">
        <div class="form-group">
          <label>Start Time (YYYY-MM-DD hh:mm:ss)</label>
          <input 
            v-model="formData.start" 
            type="text" 
            placeholder="2026-10-24 09:00:00" 
            required 
          />
        </div>

        <div class="form-group">
          <label>End Time (YYYY-MM-DD hh:mm:ss)</label>
          <input 
            v-model="formData.end" 
            type="text" 
            placeholder="2026-10-24 17:00:00" 
            required 
          />
        </div>

        <div class="button-group">
          <button type="submit" class="success">
            {{ isEditMode ? 'Update' : 'Create' }}
          </button>
          <button v-if="isEditMode" type="button" @click="deleteWorkingTime" class="danger">
            Delete
          </button>
          <button type="button" @click="$emit('close')" class="secondary">
            Cancel
          </button>
        </div>
      </form>

      <p v-if="message" :class="['message', messageType]">{{ message }}</p>
    </div>
  </div>
</template>

<script>
import api from '../services/api';

export default {
  name: 'WorkingTime',
  props: {
    userId: {
      type: [Number, String],
      required: true
    },
    item: {
      type: Object,
      default: null
    }
  },
  emits: ['saved', 'close'],
  data() {
    return {
      formData: {
        id: null,
        start: '',
        end: ''
      },
      message: '',
      messageType: 'info'
    };
  },
  computed: {
    isEditMode() {
      return !!(this.item && this.item.id);
    }
  },
  mounted() {
    if (this.item) {
      this.formData = {
        id: this.item.id,
        start: this.item.start || '',
        end: this.item.end || ''
      };
    } else {
      const now = new Date();
      this.formData.start = this.formatDate(now);
      this.formData.end = this.formatDate(new Date(now.getTime() + 8 * 60 * 60 * 1000));
    }
  },
  methods: {
    formatDate(d) {
      const pad = (n) => String(n).padStart(2, '0');
      return `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())} ${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`;
    },
    notify(text, type = 'info') {
      this.message = text;
      this.messageType = type;
      setTimeout(() => { this.message = ''; }, 3000);
    },
    handleSubmit() {
      if (this.isEditMode) {
        this.updateWorkingTime();
      } else {
        this.createWorkingTime();
      }
    },
    async createWorkingTime() {
      try {
        const payload = {
          working_time: {
            start: this.formData.start,
            end: this.formData.end
          }
        };
        await api.post(`/workingtimes/${this.userId}`, payload);
        this.$emit('saved');
      } catch (err) {
        this.notify('Failed to create record.', 'danger');
      }
    },
    async updateWorkingTime() {
      try {
        const payload = {
          working_time: {
            start: this.formData.start,
            end: this.formData.end
          }
        };
        await api.put(`/workingtimes/${this.formData.id}`, payload);
        this.$emit('saved');
      } catch (err) {
        this.notify('Failed to update record.', 'danger');
      }
    },
    async deleteWorkingTime() {
      if (!confirm('Are you sure you want to delete this working time?')) return;
      try {
        await api.delete(`/workingtimes/${this.formData.id}`);
        this.$emit('saved');
      } catch (err) {
        this.notify('Failed to delete record.', 'danger');
      }
    }
  }
};
</script>

<style scoped>
.working-time-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
.modal-card {
  background: #fff;
  padding: 24px;
  border-radius: 8px;
  width: 420px;
  max-width: 90%;
  box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}
.form {
  display: flex;
  flex-direction: column;
  gap: 14px;
  margin-top: 12px;
}
.form-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
input {
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.button-group {
  display: flex;
  gap: 8px;
  margin-top: 10px;
}
button {
  padding: 8px 14px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
}
button.success { background-color: #10b981; color: white; }
button.danger { background-color: #ef4444; color: white; }
button.secondary { background-color: #6b7280; color: white; }
.message {
  margin-top: 10px;
  padding: 8px;
  border-radius: 4px;
  font-size: 13px;
}
.message.danger { background: #fee2e2; color: #991b1b; }
.message.info { background: #e0f2fe; color: #075985; }
</style>