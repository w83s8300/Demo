<template>
  <div class="add-teacher">
    <h2>新增老師</h2>
    <form @submit.prevent="addTeacher">
      <div class="form-group">
        <label for="name">姓名:</label>
        <input type="text" id="name" v-model="teacher.name" required>
      </div>
      <div class="form-group">
        <label for="email">電子郵件:</label>
        <input type="email" id="email" v-model="teacher.email">
      </div>
      <div class="form-group">
        <label for="phone">電話:</label>
        <input type="tel" id="phone" v-model="teacher.phone">
      </div>
      <div class="form-group">
        <label for="experience_years">教學經驗 (年):</label>
        <input type="number" id="experience_years" v-model="teacher.experience_years">
      </div>
      <div class="form-group">
        <label for="bio">簡介:</label>
        <textarea id="bio" v-model="teacher.bio"></textarea>
      </div>
      <div class="form-group">
        <label for="hourly_rate">時薪:</label>
        <input type="number" id="hourly_rate" v-model="teacher.hourly_rate">
      </div>
      <div class="form-group">
        <label>專長風格:</label>
        <div v-for="style in styles" :key="style.id">
          <input type="checkbox" :id="'style-' + style.id" :value="style.id" v-model="teacher.style_ids">
          <label :for="'style-' + style.id">{{ style.name }}</label>
        </div>
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
      teacher: {
        name: '',
        email: '',
        phone: '',
        experience_years: null,
        bio: '',
        hourly_rate: null,
        style_ids: []
      },
      styles: []
    };
  },
  created() {
    this.fetchStyles();
  },
  methods: {
    fetchStyles() {
      axios.get('http://localhost:8001/api/styles')
        .then(response => {
          this.styles = response.data.styles;
        })
        .catch(error => {
          console.error('獲取風格列表失敗:', error);
        });
    },
    addTeacher() {
      axios.post('http://localhost:8001/api/teachers', this.teacher)
        .then(response => {
          console.log(response.data);
          alert('老師新增成功！');
          // 清空表單
          this.teacher = {
            name: '',
            email: '',
            phone: '',
            experience_years: null,
            bio: '',
            hourly_rate: null,
            style_ids: []
          };
        })
        .catch(error => {
          console.error(error);
          alert('新增老師失敗！');
        });
    }
  }
};
</script>

<style scoped>
.add-teacher {
  padding: 20px;
}
.form-group {
  margin-bottom: 15px;
}
label {
  display: block;
  margin-bottom: 5px;
}
input, textarea {
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
