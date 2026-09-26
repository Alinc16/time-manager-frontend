<template>
  <div class="dashboard-wrapper">
    <!-- Header -->
    <header class="app-header">
      <div class="header-content">
        <h1>⏱️ Time Manager Dashboard</h1>
        <div v-if="currentUser" class="current-user-tag">
          Active User: <strong>{{ currentUser.username }}</strong> (#{{ currentUser.id }})
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="dashboard-content">
      <!-- Top Row: User & Clock -->
      <section class="top-row">
        <User @user-selected="handleUserSelected" />
        <ClockManager 
          :user-id="currentUserId" 
          @clock-updated="handleClockUpdated" 
        />
      </section>

      <!-- Bottom Row: Working Times & Charts -->
      <section class="bottom-row">
        <WorkingTimes 
          ref="workingTimesComp"
          :user-id="currentUserId" 
          @times-updated="handleTimesUpdated" 
        />

        <ChartManager 
          :user-id="currentUserId" 
          :working-times="currentWorkingTimes" 
        />
      </section>
    </main>
  </div>
</template>

<script>
import User from './components/User.vue';
import ClockManager from './components/ClockManager.vue';
import WorkingTimes from './components/WorkingTimes.vue';
import ChartManager from './components/ChartManager.vue';

export default {
  name: 'App',
  components: {
    User,
    ClockManager,
    WorkingTimes,
    ChartManager
  },
  data() {
    return {
      currentUser: null,
      currentWorkingTimes: []
    };
  },
  computed: {
    currentUserId() {
      return this.currentUser ? this.currentUser.id : null;
    }
  },
  methods: {
    handleUserSelected(user) {
      this.currentUser = user;
      if (!user) {
        this.currentWorkingTimes = [];
      }
    },
    handleTimesUpdated(times) {
      this.currentWorkingTimes = times;
    },
    handleClockUpdated() {
      if (this.$refs.workingTimesComp) {
        this.$refs.workingTimesComp.getWorkingTimes();
      }
    }
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
}

body {
  background-color: #f3f4f6;
  color: #1f2937;
}

.dashboard-wrapper {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background-color: #1e293b;
  color: #ffffff;
  padding: 16px 32px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.header-content {
  max-width: 1300px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-content h1 {
  font-size: 20px;
  font-weight: 700;
}

.current-user-tag {
  background-color: #334155;
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 14px;
}

.dashboard-content {
  max-width: 1300px;
  margin: 24px auto;
  padding: 0 16px;
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.top-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 20px;
}

.bottom-row {
  display: flex;
  flex-direction: column;
  gap: 20px;
}
</style>