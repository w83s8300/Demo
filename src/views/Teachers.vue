<template>
  <div class="teachers">
    <h2>老師列表</h2>
    <div class="teacher-list">
      <div v-for="teacher in teachers" :key="teacher.id" class="teacher-card">
        <h3>{{ teacher.name }}</h3>
        <p><strong>電子郵件:</strong> {{ teacher.email }}</p>
        <p><strong>電話:</strong> {{ teacher.phone }}</p>
        <p><strong>經驗:</strong> {{ teacher.experience_years }} 年</p>
        <p><strong>簡介:</strong> {{ teacher.bio }}</p>
        <p><strong>時薪:</strong> {{ teacher.hourly_rate }}</p>
        <p><strong>專長風格:</strong></p>
        <ul>
          <li v-for="style in teacher.styles" :key="style.id">{{ style.name }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      teachers: []
    };
  },
  created() {
    this.fetchTeachers();
  },
  methods: {
    fetchTeachers() {
      axios.get('http://localhost:8001/api/teachers')
        .then(response => {
          this.teachers = response.data.teachers;
        })
        .catch(error => {
          console.error('獲取老師列表失敗:', error);
        });
    }
  }
};
</script>

<style scoped>
.teachers {
  padding: 20px;
}
.teacher-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}
.teacher-card {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 5px;
}
.teacher-card h3 {
  margin-top: 0;
}
.teacher-card ul {
  padding-left: 20px;
}
</style>
