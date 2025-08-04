<template>
  <div class="add-course">
    <h2>新增課程</h2>
    <form @submit.prevent="addCourse">
      <div class="form-group">
        <label for="name">課程名稱:</label>
        <input type="text" id="name" v-model="course.name" required>
      </div>
      <div class="form-group">
        <label for="description">課程描述:</label>
        <textarea id="description" v-model="course.description"></textarea>
      </div>
      <div class="form-group">
        <label for="level">難度級別:</label>
        <input type="text" id="level" v-model="course.level" required>
      </div>
      <div class="form-group">
        <label for="style_id">風格:</label>
        <select id="style_id" v-model="course.style_id">
          <option :value="null">無</option>
          <option v-for="style in styles" :key="style.id" :value="style.id">
            {{ style.name }}
          </option>
        </select>
      </div>
      <div class="form-group">
        <label for="duration_minutes">課程時長 (分鐘):</label>
        <input type="number" id="duration_minutes" v-model="course.duration_minutes">
      </div>
      <div class="form-group">
        <label for="max_students">人數上限:</label>
        <input type="number" id="max_students" v-model="course.max_students">
      </div>
      <div class="form-group">
        <label for="price">價格:</label>
        <input type="number" id="price" v-model="course.price">
      </div>
      <div class="form-group">
        <label for="teacher_id">授課老師:</label>
        <select id="teacher_id" v-model="course.teacher_id">
          <option :value="null">無</option>
          <option v-for="teacher in teachers" :key="teacher.id" :value="teacher.id">
            {{ teacher.name }}
          </option>
        </select>
      </div>
      <button type="submit">新增</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      course: {
        name: '',
        description: '',
        level: '',
        style_id: null,
        duration_minutes: 60,
        max_students: 15,
        price: null,
        teacher_id: null
      },
      teachers: [],
      styles: []
    };
  },
  created() {
    this.fetchTeachers();
    this.fetchStyles();
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
    },
    fetchStyles() {
      axios.get('http://localhost:8001/api/styles')
        .then(response => {
          this.styles = response.data.styles;
        })
        .catch(error => {
          console.error('獲取風格列表失敗:', error);
        });
    },
    addCourse() {
      axios.post('http://localhost:8001/api/courses', this.course)
        .then(response => {
          console.log(response.data);
          alert('課程新增成功！');
          // 清空表單
          this.course = {
            name: '',
            description: '',
            level: '',
            style_id: null,
            duration_minutes: 60,
            max_students: 15,
            price: null,
            teacher_id: null
          };
        })
        .catch(error => {
          console.error(error);
          alert('新增課程失敗！');
        });
    }
  }
};
</script>

<style scoped>
.add-course {
  padding: 20px;
}
.form-group {
  margin-bottom: 15px;
}
label {
  display: block;
  margin-bottom: 5px;
}
input, textarea, select {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
}
button {
  padding: 10px 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}
</style>
