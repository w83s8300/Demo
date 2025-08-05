<template>
  <div class="schedules">
    <h2>課程時間表</h2>
    <div class="schedule-list">
      <div v-for="schedule in schedules" :key="schedule.id" class="schedule-card">
        <h3>{{ schedule.course_name }}</h3>
        <p><strong>老師:</strong> {{ schedule.teacher_name }}</p>
        <p><strong>教室:</strong> {{ schedule.room_name }}</p>
        <p><strong>日期:</strong> {{ schedule.schedule_date }} ({{ schedule.day_of_week }})</p>
        <p><strong>時間:</strong> {{ schedule.start_time }} - {{ schedule.end_time }}</p>
        <p><strong>難度:</strong> {{ schedule.level }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      schedules: []
    };
  },
  created() {
    this.fetchSchedules();
  },
  methods: {
    fetchSchedules() {
      axios.get('http://localhost:8001/api/schedules')
        .then(response => {
          this.schedules = response.data.schedules;
        })
        .catch(error => {
          console.error('獲取課程時間表失敗:', error);
        });
    }
  }
};
</script>

<style scoped>
.schedules {
  padding: 20px;
}
.schedule-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}
.schedule-card {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 5px;
}
.schedule-card h3 {
  margin-top: 0;
}
</style>
