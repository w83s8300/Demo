<template>
  <div class="schedules">
    <!-- 標題和新增按鈕 -->
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h2>課程時間表管理</h2>
      <button @click="addNewSchedule" class="btn btn-success">新增課程時間表</button>
    </div>

    <!-- 月份選擇器 -->
    <div class="container my-4">
      <div class="d-flex justify-content-center mb-4">
        <input 
          type="month" 
          v-model="selectedMonth" 
          class="form-control w-auto mx-2" 
          @change="onMonthChange"
        />
        <span class="mx-2 align-self-center">
          {{ formatMonthRange(monthStartDate, monthEndDate) }}
        </span>
      </div>

      <!-- 舞蹈類型篩選器 -->
      <div class="d-flex justify-content-center flex-wrap mb-4 filter-buttons">
        <button
          v-for="type in danceTypes"
          :key="type"
          @click="selectedDanceType = type"
          :class="{ 'active': selectedDanceType === type }"
          class="btn btn-outline-primary m-2"
        >
          {{ type }}
        </button>
      </div>

      <!-- 桌面版表格 -->
      <div class="table-responsive d-none d-lg-block">
        <table class="table table-bordered table-striped text-center schedule-table">
          <thead class="table-dark">
            <tr>
              <th>時間</th>
              <th v-for="(day, index) in daysOfWeek" :key="index">
                {{ day }}<br/>
                <small>{{ getDateForDay(index) }}</small>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(timeSlot, timeIndex) in timeSlots" :key="timeIndex">
              <td class="time-slot">{{ timeSlot }}</td>
              <td v-for="(day, dayIndex) in daysOfWeek" :key="dayIndex">
                <div
                  v-for="(schedule, scheduleIndex) in getFilteredSchedules(timeSlot, day)"
                  :key="scheduleIndex"
                  :class="getLessonClass(schedule.style_name)"
                  class="schedule-cell"
                  @click="editSchedule(schedule.id)"
                  style="cursor: pointer"
                >
                  <p class="schedule-name">{{ schedule.course_name }}</p>
                  <p class="schedule-date">{{ formatScheduleDate(schedule.schedule_date) }}</p>
                  <p class="schedule-teacher">{{ schedule.teacher_name }}</p>
                  <p class="schedule-level">{{ schedule.level }}</p>
                  <div class="schedule-actions">
                    <button @click.stop="deleteSchedule(schedule.id)" class="btn btn-sm btn-danger">刪除</button>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- 手機版卡片 -->
      <div class="d-lg-none">
        <template v-for="(day, dayIndex) in daysOfWeek" :key="dayIndex">
          <div v-if="getSchedulesForDay(day).length > 0" class="mb-4">
            <h3 class="text-center">{{ day }} {{ getDateForDay(dayIndex) }}</h3>
            <div 
              v-for="(schedule, scheduleIndex) in getSchedulesForDay(day)" 
              :key="scheduleIndex" 
              :class="getLessonClass(schedule.style_name)" 
              class="schedule-card-mobile"
              @click="editSchedule(schedule.id)"
              style="cursor: pointer"
            >
              <div class="schedule-card-mobile-time">{{ formatTime(schedule.start_time) }} - {{ formatTime(schedule.end_time) }}</div>
              <div class="schedule-card-mobile-details">
                <p class="schedule-name">{{ schedule.course_name }}</p>
                <p class="schedule-date">{{ formatScheduleDate(schedule.schedule_date) }}</p>
                <p class="schedule-teacher">{{ schedule.teacher_name }}</p>
                <p class="schedule-level">{{ schedule.level }}</p>
                <div class="schedule-actions">
                  <button @click.stop="deleteSchedule(schedule.id)" class="btn btn-sm btn-danger">刪除</button>
                </div>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>

    <!-- Bootstrap Modal for adding/editing schedule -->
    <div class="modal fade" id="editScheduleModal" tabindex="-1" aria-labelledby="editScheduleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="editScheduleModalLabel">{{ editingScheduleId ? '編輯課程時間表' : '新增課程時間表' }}</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <AddSchedule v-if="showEditModal" :scheduleId="editingScheduleId" :initialDate="selectedDateForAdd" @schedule-updated="handleScheduleUpdated" @close-modal="closeEditModal" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import AddSchedule from './AddSchedule.vue';
import { Modal } from 'bootstrap';

export default {
  components: {
    AddSchedule
  },
  data() {
    return {
      schedules: [],
      showEditModal: false,
      editingScheduleId: null,
      editModal: null,
      selectedDateForAdd: null,
      // 表格相關資料
      danceTypes: [], // 從後端取得
      selectedDanceType: '全部',
      timeSlots: [
        '09:00 - 10:00', '10:00 - 11:00', '11:00 - 12:00',
        '14:00 - 15:00', '15:00 - 16:00', '18:00 - 19:00',
        '19:00 - 20:00', '20:00 - 21:00', '21:00 - 22:00'
      ],
      daysOfWeek: ['週一', '週二', '週三', '週四', '週五', '週六', '週日'],
      selectedMonth: '',
      monthStartDate: null,
      monthEndDate: null,
      monthDates: [] // 儲存該月的所有日期
    };
  },
  computed: {
    // 篩選後的課程
    filteredSchedules() {
      return this.schedules
        .filter(schedule => {
          const matchesType = this.selectedDanceType === '全部' || schedule.style_name === this.selectedDanceType;
          return matchesType;
        })
        .sort((a, b) => {
          // 先按日期排序，再按時間排序
          const dateCompare = new Date(a.schedule_date) - new Date(b.schedule_date);
          if (dateCompare !== 0) return dateCompare;
          return a.start_time.localeCompare(b.start_time);
        });
    },
    // 保留原有的 attributes 以防需要
    attributes() {
      return this.schedules.map(schedule => ({
        key: schedule.id,
        dates: new Date(schedule.schedule_date),
        dot: {
          color: this.getScheduleColor(schedule.level),
          class: 'v-calendar-dot',
        },
        popover: {
          label: `${schedule.course_name} (${schedule.start_time}-${schedule.end_time}) - ${schedule.teacher_name}`,
        },
        customData: schedule,
      }));
    },
  },
  mounted() {
    this.editModal = new Modal(document.getElementById('editScheduleModal'));
    document.getElementById('editScheduleModal').addEventListener('hidden.bs.modal', () => {
      this.showEditModal = false;
      this.editingScheduleId = null;
      this.selectedDateForAdd = null;
    });
    this.initializeCurrentMonth();
    this.fetchDanceTypes();
  },
  created() {
    this.fetchSchedules();
  },
  methods: {
    async fetchDanceTypes() {
      try {
        const response = await axios.get('http://localhost:8001/api/styles');
        if (response.data.success) {
          this.danceTypes = ['全部', ...response.data.styles.map(s => s.name)];
        } else {
          this.danceTypes = ['全部'];
        }
      } catch (error) {
        console.error('獲取舞蹈類型失敗:', error);
        this.danceTypes = ['全部'];
      }
    },
    fetchSchedules() {
      let url = 'http://localhost:8001/api/schedules';
      
      // 如果有設定月份範圍，添加查詢參數
      if (this.monthStartDate && this.monthEndDate) {
        const start_date = this.formatDateForAPI(this.monthStartDate);
        const end_date = this.formatDateForAPI(this.monthEndDate);
        url += `?start_date=${start_date}&end_date=${end_date}`;
      }

      axios.get(url)
        .then(response => {
          if (response.data.success) {
            this.schedules = response.data.schedules;
            this.generateTimeSlots(this.schedules);
          } else {
            this.schedules = [];
          }
        })
        .catch(error => {
          console.error('獲取課程時間表失敗:', error);
          this.schedules = [];
        });
    },
    initializeCurrentMonth() {
      // 設定為當前月份
      const today = new Date();
      const year = today.getFullYear();
      const month = String(today.getMonth() + 1).padStart(2, '0');
      this.selectedMonth = `${year}-${month}`;
      this.calculateMonthDates();
    },
    calculateMonthDates() {
      if (!this.selectedMonth) return;
      
      const [year, month] = this.selectedMonth.split('-');
      
      // 該月的第一天
      this.monthStartDate = new Date(parseInt(year), parseInt(month) - 1, 1);
      
      // 該月的最後一天
      this.monthEndDate = new Date(parseInt(year), parseInt(month), 0);
      
      // 計算該月的所有日期
      this.monthDates = [];
      for (let d = new Date(this.monthStartDate); d <= this.monthEndDate; d.setDate(d.getDate() + 1)) {
        this.monthDates.push(new Date(d));
      }
    },
    onMonthChange() {
      this.calculateMonthDates();
      this.fetchSchedules();
    },
    formatMonthRange(startDate, endDate) {
      if (!startDate || !endDate) return '';
      const year = startDate.getFullYear();
      const month = startDate.getMonth() + 1;
      return `${year}年${month}月`;
    },
    getDateForDay(dayIndex) {
      // 對於月份視圖，顯示該月份中該星期幾的所有日期
      if (!this.monthDates.length) return '';
      
      const dayNames = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
      const chineseDays = ['週日', '週一', '週二', '週三', '週四', '週五', '週六'];
      const targetDay = chineseDays[dayIndex];
      
      // 找出該月份中所有符合該星期幾的日期
      const datesForDay = this.monthDates.filter(date => {
        const dayName = dayNames[date.getDay()];
        return this.getDayText(dayName) === targetDay;
      });
      
      if (datesForDay.length === 0) return '';
      
      // 顯示日期列表，例如：1,8,15,22,29
      return datesForDay.map(date => date.getDate()).join(',');
    },
    formatDateForInput(date) {
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    },
    formatDateForAPI(date) {
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    },
    formatTime(timeString) {
      // 將 "14:50:00" 格式轉換為 "14:50"
      if (timeString && timeString.includes(':')) {
        const parts = timeString.split(':');
        return `${parts[0]}:${parts[1]}`;
      }
      return timeString;
    },
    generateTimeSlots(schedules) {
      // 從API資料中收集所有不重複的時間段
      const timeSlotSet = new Set();
      
      schedules.forEach(schedule => {
        const startTime = this.formatTime(schedule.start_time);
        const endTime = this.formatTime(schedule.end_time);
        const timeSlot = `${startTime} - ${endTime}`;
        timeSlotSet.add(timeSlot);
      });
      
      // 將時間段轉換為陣列並排序
      const newTimeSlots = Array.from(timeSlotSet).sort((a, b) => {
        const timeA = a.split(' - ')[0];
        const timeB = b.split(' - ')[0];
        return timeA.localeCompare(timeB);
      });
      
      // 如果有新的時間段，更新 timeSlots
      if (newTimeSlots.length > 0) {
        this.timeSlots = newTimeSlots;
      }
    },
    findMatchingTimeSlot(startTime, endTime) {
      // 格式化時間
      const formattedStart = this.formatTime(startTime);
      const formattedEnd = this.formatTime(endTime);
      const targetTimeSlot = `${formattedStart} - ${formattedEnd}`;
      
      // 在現有的 timeSlots 中尋找匹配的時間段
      const matchingSlot = this.timeSlots.find(slot => slot === targetTimeSlot);
      
      if (matchingSlot) {
        return matchingSlot;
      }
      
      // 如果找不到完全匹配的，返回格式化後的時間段
      return targetTimeSlot;
    },
    getDayText(dayOfWeek) {
      // 英文轉中文
      const map = {
        'Monday': '週一', 'Tuesday': '週二', 'Wednesday': '週三', 'Thursday': '週四', 
        'Friday': '週五', 'Saturday': '週六', 'Sunday': '週日'
      };
      return map[dayOfWeek] || dayOfWeek;
    },
    formatScheduleDate(dateString) {
      // 格式化日期顯示，例如：8/15
      const date = new Date(dateString);
      return `${date.getMonth() + 1}/${date.getDate()}`;
    },
    getFilteredSchedules(time, day) {
      const timeSlot = this.findMatchingTimeSlot(time.split(' - ')[0], time.split(' - ')[1]);
      return this.schedules.filter(schedule => {
        const scheduleTimeSlot = this.findMatchingTimeSlot(schedule.start_time, schedule.end_time);
        const matchesTime = scheduleTimeSlot === time;
        const matchesDay = this.getDayText(schedule.day_of_week) === day;
        const matchesType = this.selectedDanceType === '全部' || schedule.style_name === this.selectedDanceType;
        return matchesTime && matchesDay && matchesType;
      });
    },
    getSchedulesForDay(day) {
      return this.schedules
        .filter(schedule => {
          const matchesDay = this.getDayText(schedule.day_of_week) === day;
          const matchesType = this.selectedDanceType === '全部' || schedule.style_name === this.selectedDanceType;
          return matchesDay && matchesType;
        })
        .sort((a, b) => a.start_time.localeCompare(b.start_time));
    },
    getLessonClass(type) {
      // Map dance types to CSS classes for background colors
      const classMap = {
        'Hip Hop': 'bg-hiphop',
        'Jazz': 'bg-jazz',
        'K-POP': 'bg-kpop',
        'Locking': 'bg-locking',
        'Waacking': 'bg-waacking',
        'Breaking': 'bg-breaking',
        'Popping': 'bg-popping',
        'Heels': 'bg-heels',
        'Contemporary': 'bg-contemporary',
        'Vogue': 'bg-vogue',
        'Street Jazz': 'bg-jazz' // Street Jazz uses Jazz color
      };
      return classMap[type] || 'bg-secondary';
    },
    getScheduleColor(level) {
      switch (level) {
        case '初級': return 'blue';
        case '中級': return 'green';
        case '高級': return 'red';
        default: return 'gray';
      }
    },
    addNewSchedule() {
      this.editingScheduleId = null;
      this.selectedDateForAdd = null; // 新增時不預設日期
      this.showEditModal = true;
      this.editModal.show();
    },
    editSchedule(id) {
      this.editingScheduleId = id;
      this.selectedDateForAdd = null; // 編輯時不預設日期
      this.showEditModal = true;
      this.editModal.show();
    },
    deleteSchedule(id) {
      if (confirm('確定要刪除這個課程時間表嗎？')) {
        axios.delete(`http://localhost:8001/api/schedules/${id}`)
          .then(() => {
            this.fetchSchedules();
          })
          .catch(error => {
            console.error('刪除課程時間表失敗:', error);
          });
      }
    },
    handleScheduleUpdated() {
      this.fetchSchedules();
      this.editModal.hide();
    },
    closeEditModal() {
      this.editModal.hide();
    },
    onDayClick(day) {
      // 點擊日期時，預填日期並開啟新增視窗
      this.selectedDateForAdd = day.id; // day.id 格式為 YYYY-MM-DD
      this.addNewSchedule();
    },
  }
};
</script>

<style scoped>
.schedules {
  padding: 20px;
}

/* 表格樣式 */
.schedule-table {
  background-color: #ffffff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  overflow: hidden;
}

.schedule-table th {
  background-color: #343a40;
  color: white;
  border: none;
  padding: 15px 8px;
  font-weight: 600;
}

.schedule-table td {
  vertical-align: middle;
  padding: 5px;
  min-height: 80px;
}

.time-slot {
  background-color: #f8f9fa;
  font-weight: 600;
  writing-mode: vertical-rl;
  text-orientation: mixed;
  min-width: 60px;
}

/* 課程格子樣式 */
.schedule-cell {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 8px;
  padding: 8px;
  margin: 2px;
  transition: all 0.3s ease;
  position: relative;
  min-height: 60px;
}

.schedule-cell:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.schedule-name {
  font-weight: bold;
  margin: 0 0 4px 0;
  font-size: 0.9rem;
}

.schedule-date {
  margin: 0 0 4px 0;
  font-size: 0.75rem;
  opacity: 0.8;
  font-style: italic;
}

.schedule-teacher {
  margin: 0 0 4px 0;
  font-size: 0.8rem;
  opacity: 0.9;
}

.schedule-level {
  margin: 0 0 4px 0;
  font-size: 0.7rem;
  background: rgba(255, 255, 255, 0.2);
  padding: 2px 6px;
  border-radius: 12px;
  display: inline-block;
}

.schedule-actions {
  margin-top: 4px;
}

.schedule-actions .btn {
  font-size: 0.7rem;
  padding: 2px 6px;
}

/* 舞蹈類型背景色 */

/* Reusing colors from Classes.vue for consistency */
.bg-hiphop { background: #d9534f ; color: black !important; }
.bg-jazz { background: #5bc0de ; color: black !important; }
.bg-kpop { background: #5cb85c !important; color: black !important; }
.bg-locking { background: #f0ad4e !important; color: black !important; }
.bg-waacking { background: #9b59b6 !important; color: black !important; }
.bg-breaking { background: #e74c3c !important; color: black !important; }
.bg-popping { background: #3498db !important; color: black !important; }
.bg-heels { background: #d63384 !important; color: black !important; }
.bg-contemporary { background: #1abc9c !important; color: black !important; }
.bg-vogue { background: #f1c40f !important; color: black !important; }


/* 手機版卡片 */
.schedule-card-mobile {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 8px;
  padding: 12px;
  margin: 8px 0;
  display: flex;
  transition: all 0.3s ease;
}

.schedule-card-mobile:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.schedule-card-mobile-time {
  min-width: 80px;
  font-weight: bold;
  font-size: 0.9rem;
  margin-right: 12px;
}

.schedule-card-mobile-details {
  flex: 1;
}

/* 篩選按鈕 */
.filter-buttons .btn {
  border-radius: 20px;
  padding: 8px 16px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.filter-buttons .btn.active {
  background-color: #007bff;
  color: white;
  border-color: #007bff;
}

.filter-buttons .btn:hover {
  transform: translateY(-1px);
}

/* 響應式設計 */
@media (max-width: 768px) {
  .schedules {
    padding: 10px;
  }
  
  .filter-buttons .btn {
    font-size: 0.8rem;
    padding: 6px 12px;
    margin: 4px;
  }
}

/* Modal styles (保留原有樣式) */
.modal-content {
  border-radius: 0.5rem;
}

.modal-header {
  border-bottom: 1px solid #dee2e6;
}

.modal-footer {
  border-top: 1px solid #dee2e6;
}
</style>
