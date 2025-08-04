<template>
  <div class="students">
    <h2>學生列表</h2>
    <div class="student-list">
      <div v-for="student in students" :key="student.id" class="student-card">
        <h3>{{ student.name }}</h3>
        <p><strong>電子郵件:</strong> {{ student.email }}</p>
        <p><strong>電話:</strong> {{ student.phone }}</p>
        <p><strong>年齡:</strong> {{ student.age }}</p>
        <p><strong>緊急聯絡人:</strong> {{ student.emergency_contact }}</p>
        <p><strong>緊急聯絡人電話:</strong> {{ student.emergency_phone }}</p>
        <p><strong>醫療備註:</strong> {{ student.medical_notes }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      students: []
    };
  },
  created() {
    this.fetchStudents();
  },
  methods: {
    fetchStudents() {
      axios.get('http://localhost:8001/api/students')
        .then(response => {
          this.students = response.data.students;
        })
        .catch(error => {
          console.error('獲取學生列表失敗:', error);
        });
    }
  }
};
</script>

<style scoped>
.students {
  padding: 20px;
}
.student-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}
.student-card {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 5px;
}
.student-card h3 {
  margin-top: 0;
}
</style>
