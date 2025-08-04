<template>
  <div class="rooms">
    <h2>教室列表</h2>
    <div class="room-list">
      <div v-for="room in rooms" :key="room.id" class="room-card">
        <h3>{{ room.name }}</h3>
        <p><strong>容量:</strong> {{ room.capacity }}</p>
        <p><strong>設備:</strong> {{ room.equipment }}</p>
        <p><strong>描述:</strong> {{ room.description }}</p>
        <p><strong>時租費率:</strong> {{ room.hourly_rate }}</p>
        <p><strong>狀態:</strong> {{ room.is_available ? '可用' : '不可用' }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      rooms: []
    };
  },
  created() {
    this.fetchRooms();
  },
  methods: {
    fetchRooms() {
      axios.get('http://localhost:8001/api/rooms')
        .then(response => {
          this.rooms = response.data.rooms;
        })
        .catch(error => {
          console.error('獲取教室列表失敗:', error);
        });
    }
  }
};
</script>

<style scoped>
.rooms {
  padding: 20px;
}
.room-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}
.room-card {
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 5px;
}
.room-card h3 {
  margin-top: 0;
}
</style>
