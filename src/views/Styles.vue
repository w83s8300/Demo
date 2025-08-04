<template>
  <div class="styles">
    <h2>風格管理</h2>
    
    <!-- 新增風格表單 -->
    <form @submit.prevent="addStyle" class="add-style-form">
      <h3>新增風格</h3>
      <div class="form-group">
        <label for="name">風格名稱:</label>
        <input type="text" id="name" v-model="newStyle.name" required>
      </div>
      <div class="form-group">
        <label for="description">風格描述:</label>
        <textarea id="description" v-model="newStyle.description"></textarea>
      </div>
      <button type="submit">新增</button>
    </form>

    <!-- 風格列表 -->
    <div class="style-list">
      <h3>風格列表</h3>
      <ul>
        <li v-for="style in styles" :key="style.id">
          <h4>{{ style.name }}</h4>
          <p>{{ style.description }}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      styles: [],
      newStyle: {
        name: '',
        description: ''
      }
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
    addStyle() {
      axios.post('http://localhost:8001/api/styles', this.newStyle)
        .then(response => {
          console.log(response.data);
          alert('風格新增成功！');
          this.fetchStyles(); // 重新獲取列表
          // 清空表單
          this.newStyle = {
            name: '',
            description: ''
          };
        })
        .catch(error => {
          console.error(error);
          alert('新增風格失敗！');
        });
    }
  }
};
</script>

<style scoped>
.styles {
  padding: 20px;
}
.add-style-form, .style-list {
  margin-bottom: 30px;
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
.style-list ul {
  list-style-type: none;
  padding: 0;
}
.style-list li {
  border-bottom: 1px solid #ccc;
  padding: 10px 0;
}
</style>
