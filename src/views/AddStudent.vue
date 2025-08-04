<template>
  <div class="add-student">
    <h2>新增學生</h2>
    <form @submit.prevent="addStudent">
      <div class="form-group">
        <label for="name">姓名:</label>
        <input type="text" id="name" v-model="student.name" required>
      </div>
      <div class="form-group">
        <label for="email">電子郵件:</label>
        <input type="email" id="email" v-model="student.email" required>
      </div>
      <div class="form-group">
        <label for="phone">電話:</label>
        <input type="tel" id="phone" v-model="student.phone">
      </div>
      <div class="form-group">
        <label for="age">年齡:</label>
        <input type="number" id="age" v-model="student.age">
      </div>
      <div class="form-group">
        <label for="emergency_contact">緊急聯絡人:</label>
        <input type="text" id="emergency_contact" v-model="student.emergency_contact">
      </div>
      <div class="form-group">
        <label for="emergency_phone">緊急聯絡人電話:</label>
        <input type="tel" id="emergency_phone" v-model="student.emergency_phone">
      </div>
      <div class="form-group">
        <label for="medical_notes">醫療備註:</label>
        <textarea id="medical_notes" v-model="student.medical_notes"></textarea>
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
      student: {
        name: '',
        email: '',
        phone: '',
        age: null,
        emergency_contact: '',
        emergency_phone: '',
        medical_notes: ''
      }
    };
  },
  methods: {
    addStudent() {
      axios.post('http://localhost:8001/api/students', this.student)
        .then(response => {
          console.log(response.data);
          alert('學生新增成功！');
          // 清空表單
          this.student = {
            name: '',
            email: '',
            phone: '',
            age: null,
            emergency_contact: '',
            emergency_phone: '',
            medical_notes: ''
          };
        })
        .catch(error => {
          console.error(error);
          alert('新增學生失敗！');
        });
    }
  }
};
</script>

<style scoped>
.add-student {
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
