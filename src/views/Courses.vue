<template>
  <div class="courses">
    <h2>課程列表</h2>
    <div class="course-list">
      <div v-for="course in courses" :key="course.id" class="course-card">
        <h3>{{ course.name }}</h3>
        <p><strong>描述:</strong> {{ course.description }}</p>
        <p><strong>難度:</strong> {{ course.level }}</p>
        <p><strong>風格:</strong> {{ course.style_name }}</p>
        <p><strong>時長:</strong> {{ course.duration_minutes }} 分鐘</p>
        <p><strong>人數上限:</strong> {{ course.max_students }}</p>
        <p><strong>價格:</strong> {{ course.price }}</p>
        <p><strong>老師:</strong> {{ course.teacher_name }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      courses: []
    };
  },
  created() {
    this.fetchCourses();
  },
  methods: {
    fetchCourses() {
      axios.get('http://localhost:8001/api/courses')
        .then(response => {
          this.courses = response.data.courses;
        })
        .catch(error => {
          console.error('獲取課程列表失敗:', error);
        });
    }
  }
};
</script>

<style scoped>
.courses {
  padding: 20px;
}
.course-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}
.course-card {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 5px;
}
.course-card h3 {
  margin-top: 0;
}
</style>
