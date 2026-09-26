<template>
  <div class="card">
    <h2>👤 User Management</h2>

    <!-- Search User -->
    <div class="row">
      <input v-model="searchId" type="number" placeholder="Search by User ID" />
      <button @click="getUser(searchId)">Get User</button>
      <button @click="fetchAllUsers" class="secondary">Refresh All</button>
    </div>

    <!-- User Select Dropdown -->
    <div class="row" v-if="usersList.length > 0">
      <label>Existing Users:</label>
      <select v-model="selectedUserId" @change="onSelectUser">
        <option :value="null">-- Select a user --</option>
        <option v-for="u in usersList" :key="u.id" :value="u.id">
          #{{ u.id }} - {{ u.username }} ({{ u.email }})
        </option>
      </select>
    </div>

    <!-- Form -->
    <form @submit.prevent="handleSubmit" class="form">
      <div class="form-group">
        <label>Username</label>
        <input v-model="form.username" required placeholder="username" />
      </div>
      <div class="form-group">
        <label>Email</label>
        <input v-model="form.email" type="email" required placeholder="user@epitech.eu" />
      </div>

      <div class="button-group">
        <button type="submit" class="success">
          {{ isEditing ? 'Update User' : 'Create User' }}
        </button>
        <button v-if="isEditing" type="button" @click="resetForm" class="secondary">Cancel</button>
        <button v-if="isEditing" type="button" @click="deleteUser(form.id)" class="danger">
          Delete User
        </button>
      </div>
    </form>

    <p v-if="message" :class="['message', messageType]">{{ message }}</p>
  </div>
</template>

<script>
import api from '../services/api';

export default {
  name: 'User',
  emits: ['user-selected'],
  data() {
    return {
      usersList: [],
      selectedUserId: null,
      searchId: '',
      form: {
        id: null,
        username: '',
        email: ''
      },
      isEditing: false,
      message: '',
      messageType: 'info'
    };
  },
  mounted() {
    this.fetchAllUsers();
  },
  methods: {
    notify(text, type = 'info') {
      this.message = text;
      this.messageType = type;
      setTimeout(() => { this.message = ''; }, 4000);
    },
    resetForm() {
      this.form = { id: null, username: '', email: '' };
      this.isEditing = false;
    },
    onSelectUser() {
      if (this.selectedUserId) {
        this.getUser(this.selectedUserId);
      } else {
        this.resetForm();
        this.$emit('user-selected', null);
      }
    },
    async fetchAllUsers() {
      try {
        const res = await api.get('/users');
        this.usersList = res.data.data || res.data;
      } catch (err) {
        console.error('Failed to fetch users:', err);
      }
    },
    handleSubmit() {
      if (this.isEditing) {
        this.updateUser();
      } else {
        this.createUser();
      }
    },
    async getUser(id) {
      if (!id) return;
      try {
        const res = await api.get(`/users/${id}`);
        const user = res.data.data || res.data;
        this.form = { id: user.id, username: user.username, email: user.email };
        this.selectedUserId = user.id;
        this.isEditing = true;
        this.$emit('user-selected', user);
        this.notify(`User loaded: ${user.username}`, 'success');
      } catch (err) {
        this.notify('User not found!', 'danger');
      }
    },
    async createUser() {
      try {
        const payload = { user: { username: this.form.username, email: this.form.email } };
        const res = await api.post('/users', payload);
        const newUser = res.data.data || res.data;
        this.notify('User created successfully.', 'success');
        await this.fetchAllUsers();
        this.getUser(newUser.id);
      } catch (err) {
        this.notify('Failed to create user.', 'danger');
      }
    },
    async updateUser() {
      if (!this.form.id) return;
      try {
        const payload = { user: { username: this.form.username, email: this.form.email } };
        const res = await api.put(`/users/${this.form.id}`, payload);
        const updated = res.data.data || res.data;
        this.notify('User updated successfully.', 'success');
        await this.fetchAllUsers();
        this.$emit('user-selected', updated);
      } catch (err) {
        this.notify('Failed to update user.', 'danger');
      }
    },
    async deleteUser(id) {
      if (!id || !confirm('Are you sure you want to delete this user?')) return;
      try {
        await api.delete(`/users/${id}`);
        this.notify('User deleted successfully.', 'success');
        this.resetForm();
        this.selectedUserId = null;
        this.$emit('user-selected', null);
        await this.fetchAllUsers();
      } catch (err) {
        this.notify('Failed to delete user.', 'danger');
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
.row {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-bottom: 15px;
}
.form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.form-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
input, select {
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}
.button-group {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}
button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  background-color: #3b82f6;
  color: white;
  font-weight: 600;
}
button.secondary { background-color: #6b7280; }
button.success { background-color: #10b981; }
button.danger { background-color: #ef4444; }
.message {
  margin-top: 10px;
  padding: 8px;
  border-radius: 4px;
  font-size: 14px;
}
.message.success { background: #d1fae5; color: #065f46; }
.message.danger { background: #fee2e2; color: #991b1b; }
.message.info { background: #e0f2fe; color: #075985; }
</style>