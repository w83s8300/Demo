<template>
  <div class="add-schedule">
    <form @submit.prevent="submitForm">
      <div class="form-group">
        <label for="course_id">課程:</label>
        <select id="course_id" v-model="schedule.course_id" required>
          <option v-for="course in courses" :key="course.id" :value="course.id">
            {{ course.name }}
          </option>
        </select>
      </div>
      <div class="form-group">
        <label for="room_id">教室:</label>
        <select id="room_id" v-model="schedule.room_id">
          <option :value="null">無</option>
          <option v-for="room in rooms" :key="room.id" :value="room.id">
            {{ room.name }}
          </option>
        </select>
      </div>
      <div class="form-group">
        <label for="schedule_date">日期:</label>
        <input type="date" id="schedule_date" v-model="schedule.schedule_date" required>
      </div>
      <div class="form-group">
        <label for="start_time">開始時間:</label>
        <input type="time" id="start_time" v-model="schedule.start_time" required>
      </div>
      <div class="form-group">
        <label for="end_time">結束時間:</label>
        <input type="time" id="end_time" v-model="schedule.end_time" required>
      </div>
      <button type="submit" class="btn btn-primary">{{ isEditMode ? '更新' : '新增' }}</button>
      <button type="button" class="btn btn-secondary" @click="closeModal">取消</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  props: {
    scheduleId: {
      type: Number,
      default: null
    }
  },
  data() {
    return {
      schedule: {
        course_id: null,
        room_id: null,
        schedule_date: '',
        start_time: '',
        end_time: ''
      },
      courses: [],
      rooms: [],
      isEditMode: false
    };
  },
  created() {
    this.fetchCourses();
    this.fetchRooms();
    if (this.scheduleId) {
      this.isEditMode = true;
      this.fetchSchedule(this.scheduleId);
    }
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
    },
    fetchRooms() {
      axios.get('http://localhost:8001/api/rooms')
        .then(response => {
          this.rooms = response.data.rooms;
        })
        .catch(error => {
          console.error('獲取教室列表失敗:', error);
        });
    },
    fetchSchedule(id) {
      axios.get(`http://localhost:8001/api/schedules/${id}`)
        .then(response => {
          this.schedule = response.data.schedule;
          // 確保日期格式正確，以便 input type="date" 顯示
          if (this.schedule.schedule_date) {
            this.schedule.schedule_date = this.schedule.schedule_date.split('T')[0];
          }
        })
        .catch(error => {
          console.error('獲取課程時間表資料失敗:', error);
        });
    },
    submitForm() {
      if (this.isEditMode) {
        this.updateSchedule();
      } else {
        this.addSchedule();
      }
    },
    addSchedule() {
      axios.post('http://localhost:8001/api/schedules', this.schedule)
        .then(() => {
          alert('課程時間表新增成功！');
          this.$emit('schedule-updated');
        })
        .catch(error => {
          console.error(error);
          alert('新增課程時間表失敗！');
        });
    },
    updateSchedule() {
      axios.put(`http://localhost:8001/api/schedules/${this.scheduleId}`, this.schedule)
        .then(() => {
          alert('課程時間表更新成功！');
          this.$emit('schedule-updated');
        })
        .catch(error => {
          console.error(error);
          alert('更新課程時間表失敗！');
        });
    },
    closeModal() {
      this.$emit('close-modal');
    }
  }
};
</script>

<style scoped>
.add-schedule {
  padding: 20px;
}
.form-group {
  margin-bottom: 15px;
}
label {
  display: block;
  margin-bottom: 5px;
}
input, select {
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
