<template>
  <div class="schedules">
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h2>課程時間表</h2>
      <button @click="addNewSchedule" class="btn btn-success">新增課程時間表</button>
    </div>
    <table class="schedule-table">
      <thead>
        <tr>
          <th>課程名稱</th>
          <th>老師</th>
          <th>教室</th>
          <th>日期</th>
          <th>時間</th>
          <th>難度</th>
          <th>操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="schedule in schedules" :key="schedule.id">
          <td>{{ schedule.course_name }}</td>
          <td>{{ schedule.teacher_name }}</td>
          <td>{{ schedule.room_name }}</td>
          <td>{{ schedule.schedule_date }} ({{ schedule.day_of_week }})</td>
          <td>{{ schedule.start_time }} - {{ schedule.end_time }}</td>
          <td>{{ schedule.level }}</td>
          <td>
            <button @click="editSchedule(schedule.id)" class="btn btn-primary btn-sm">編輯</button>
            <button @click="deleteSchedule(schedule.id)" class="btn btn-danger btn-sm">移除</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Bootstrap Modal for adding/editing schedule -->
    <div class="modal fade" id="editScheduleModal" tabindex="-1" aria-labelledby="editScheduleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="editScheduleModalLabel">{{ editingScheduleId ? '編輯課程時間表' : '新增課程時間表' }}</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <AddSchedule v-if="showEditModal" :scheduleId="editingScheduleId" @schedule-updated="handleScheduleUpdated" @close-modal="closeEditModal" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import AddSchedule from './AddSchedule.vue'; // 引入 AddSchedule 組件
import { Modal } from 'bootstrap'; // 引入 Bootstrap Modal JS

export default {
  components: {
    AddSchedule
  },
  data() {
    return {
      schedules: [],
      showEditModal: false, // 控制 AddSchedule 組件的渲染
      editingScheduleId: null,
      editModal: null // 用於儲存 Bootstrap Modal 實例
    };
  },
  mounted() {
    // 在組件掛載後初始化 Bootstrap Modal
    this.editModal = new Modal(document.getElementById('editScheduleModal'));
    // 監聽 modal 關閉事件，以便重置狀態
    document.getElementById('editScheduleModal').addEventListener('hidden.bs.modal', () => {
      this.showEditModal = false;
      this.editingScheduleId = null;
    });
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
    },
    addNewSchedule() {
      this.editingScheduleId = null; // 設定為 null 表示新增模式
      this.showEditModal = true; // 渲染 AddSchedule 組件
      this.editModal.show(); // 顯示 Bootstrap Modal
    },
    editSchedule(id) {
      this.editingScheduleId = id;
      this.showEditModal = true; // 渲染 AddSchedule 組件
      this.editModal.show(); // 顯示 Bootstrap Modal
    },
    deleteSchedule(id) {
      if (confirm('確定要刪除這個課程時間表嗎？')) {
        axios.delete(`http://localhost:8001/api/schedules/${id}`)
          .then(() => {
            this.fetchSchedules(); // Refresh the list after deletion
          })
          .catch(error => {
            console.error('刪除課程時間表失敗:', error);
          });
      }
    },
    handleScheduleUpdated() {
      this.fetchSchedules(); // 課程時間表數據更新後刷新列表
      this.editModal.hide(); // 關閉 Bootstrap Modal
    },
    closeEditModal() {
      // 由 AddSchedule 組件發出，用於取消或錯誤時關閉 modal
      this.editModal.hide();
    }
  }
};
</script>

<style scoped>
.schedules {
  padding: 20px;
}
.schedule-table {
  width: 100%;
  border-collapse: collapse;
}
.schedule-table th, .schedule-table td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: left;
}
.schedule-table th {
  background-color: #f2f2f2;
}
.schedule-table button {
  margin-right: 5px;
}
</style>
