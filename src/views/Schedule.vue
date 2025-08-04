<template>
  <div class="schedule-page">
    <!-- 1. Banner -->
    <div class="banner">
      <img src="https://picsum.photos/1920/500?random=36" alt="Schedule Banner" class="banner-img">
      <div class="banner-text">
        <h1>課程時間表 / SCHEDULE</h1>
      </div>
    </div>

    <!-- 2. Class Schedule with Filters -->
    <div class="container my-5">
      <h2 class="text-center mb-4">完整課程時間表</h2>

      <!-- 年月選擇器 -->
      <div class="d-flex justify-content-center mb-4">
        <select v-model="selectedYear" class="form-select w-auto mx-2">
          <option v-for="year in years" :key="year" :value="year">{{ year }} 年</option>
        </select>
        <select v-model="selectedMonth" class="form-select w-auto mx-2">
          <option v-for="month in months" :key="month" :value="month">{{ month }} 月</option>
        </select>
        <button class="btn btn-primary mx-2" @click="fetchSchedule">查詢</button>
      </div>

      <!-- Dance Type Filters -->
      <div class="d-flex justify-content-center flex-wrap mb-4 filter-buttons">
        <button
          v-for="type in danceTypes"
          :key="type"
          @click="selectedDanceType = type"
          :class="{ 'active': selectedDanceType === type }"
          class="btn btn-outline-light m-2"
        >
          {{ type }}
        </button>
      </div>

      <!-- Desktop Table View -->
      <div class="table-responsive d-none d-lg-block">
        <table class="table table-bordered table-Secondary table-striped text-center schedule-table">
          <thead>
            <tr>
              <th>時間</th>
              <th>週一</th>
              <th>週二</th>
              <th>週三</th>
              <th>週四</th>
              <th>週五</th>
              <th>週六</th>
              <th>週日</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(timeSlot, timeIndex) in timeSlots" :key="timeIndex">
              <td>{{ timeSlot }}</td>
              <td v-for="(day, dayIndex) in daysOfWeek" :key="dayIndex">
                <div
                  v-for="(lesson, lessonIndex) in getFilteredLessons(timeSlot, day)"
                  :key="lessonIndex"
                  :class="getLessonClass(lesson.type)"
                  class="lesson-cell"
                  @click="openModal(lesson)"
                  style="cursor: pointer"
                >
                  <p class="lesson-name">{{ lesson.name }}</p>
                  <p class="lesson-teacher">{{ lesson.teacher }}</p>
                  <p class="lesson-level">{{ lesson.level }}</p>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Mobile Card View -->
      <div class="d-lg-none">
        <template v-for="(day, dayIndex) in daysOfWeek" :key="dayIndex">
          <div v-if="getLessonsForDay(day).length > 0" class="mb-4">
            <h3 class="text-center">{{ day }}</h3>
            <div 
              v-for="(lesson, lessonIndex) in getLessonsForDay(day)" 
              :key="lessonIndex" 
              :class="getLessonClass(lesson.type)" 
              class="lesson-card-mobile"
              @click="openModal(lesson)"
              style="cursor: pointer"
            >
              <div class="lesson-card-mobile-time">{{ lesson.time }}</div>
              <div class="lesson-card-mobile-details">
                <p class="lesson-name">{{ lesson.name }}</p>
                <p class="lesson-teacher">{{ lesson.teacher }}</p>
                <p class="lesson-level">{{ lesson.level }}</p>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>

    <!-- Lesson Details Modal -->
    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h4 class="modal-title">課程詳情</h4>
          <button type="button" class="btn-close" @click="closeModal">&times;</button>
        </div>
        <div class="modal-body" v-if="selectedLesson">
          <div class="lesson-detail-card" >
            <h5 class="lesson-name-large">{{ selectedLesson.name }}</h5>
            <div class="lesson-info">
              <div class="info-item">
                <strong>時間：</strong>{{ selectedLesson.time }}
              </div>
              <div class="info-item">
                <strong>星期：</strong>{{ selectedLesson.day }}
              </div>
              <div class="info-item">
                <strong>教師：</strong>{{ selectedLesson.teacher }}
              </div>
              <div class="info-item">
                <strong>程度：</strong>{{ selectedLesson.level }}
              </div>
              <div class="info-item">
                <strong>舞蹈類型：</strong>{{ selectedLesson.type }}
              </div>
            </div>
            <div class="lesson-description">
              <h6>課程介紹</h6>
              <p>{{ getLessonDescription(selectedLesson.type) }}</p>
            </div>
            <div class="enrollment-form">
              <h6>報名資訊</h6>
                <div class="form-group row">
                <label for="studentName" class="col-2 col-form-label">姓名：</label>
                <div class="col-10">
                  <input 
                  type="text" 
                  id="studentName" 
                  v-model="studentName" 
                  class="form-control" 
                  placeholder="請輸入您的姓名"
                  @keyup.enter="enrollLesson(selectedLesson)"
                  />
                </div>
                </div>
            </div>
            <div class="lesson-actions">
              <button class="btn btn-primary me-2" @click="enrollLesson(selectedLesson)">立即報名</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Schedule',
  data() {
    return {
      showModal: false,
      selectedLesson: null,
      studentName: '',
      danceTypes: ['全部', 'Hip Hop', 'Jazz', 'K-POP', 'Locking', 'Waacking', 'Breaking', 'Popping', 'Heels', 'Contemporary', 'Vogue'],
      selectedDanceType: '全部',
      timeSlots: [
        '09:00 - 10:00', '10:00 - 11:00', '11:00 - 12:00',
        '14:00 - 15:00', '15:00 - 16:00', '18:00 - 19:00',
        '19:00 - 20:00', '20:00 - 21:00', '21:00 - 22:00'
      ],
      daysOfWeek: ['週一', '週二', '週三', '週四', '週五', '週六', '週日'],
      classes: [], // 從後端取得
      years: [2024, 2025, 2026],
      months: [1,2,3,4,5,6,7,8,9,10,11,12],
      selectedYear: new Date().getFullYear(),
      selectedMonth: new Date().getMonth() + 1
    };
  },
  mounted() {
    this.fetchSchedule();
  },
  methods: {
    async fetchSchedule() {
      // 計算本月第一天和最後一天
      const year = this.selectedYear;
      const month = this.selectedMonth;
      const start_date = `${year}-${String(month).padStart(2, '0')}-01`;
      const end_date = `${year}-${String(month).padStart(2, '0')}-31`;
      try {
        const response = await fetch(`http://localhost:8001/api/schedules?start_date=${start_date}&end_date=${end_date}`);
        const result = await response.json();
        if (response.ok && result.success) {
          // 轉換後端資料格式為前端課表格式
          this.classes = result.schedules.map(s => ({
            time: `${s.start_time} - ${s.end_time}`,
            day: this.getDayText(s.day_of_week),
            name: s.course_name,
            teacher: s.teacher_name || '',
            level: s.level,
            type: s.type
          }));
        } else {
          this.classes = [];
        }
      } catch (error) {
        this.classes = [];
      }
    },
    getDayText(dayOfWeek) {
      // 英文轉中文
      const map = {
        'Monday': '週一', 'Tuesday': '週二', 'Wednesday': '週三', 'Thursday': '週四', 'Friday': '週五', 'Saturday': '週六', 'Sunday': '週日'
      };
      return map[dayOfWeek] || dayOfWeek;
    },
    getFilteredLessons(time, day) {
      return this.classes.filter(lesson => {
        const matchesTime = lesson.time === time;
        const matchesDay = lesson.day === day;
        const matchesType = this.selectedDanceType === '全部' || lesson.type === this.selectedDanceType;
        return matchesTime && matchesDay && matchesType;
      });
    },
    getLessonsForDay(day) {
      return this.classes
        .filter(lesson => lesson.day === day && (this.selectedDanceType === '全部' || lesson.type === this.selectedDanceType))
        .sort((a, b) => a.time.localeCompare(b.time));
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
      return classMap[type] || '';
    },
    openModal(lesson) {
      this.selectedLesson = lesson;
      this.showModal = true;
      // 防止背景滾動
      document.body.style.overflow = 'hidden';
    },
    closeModal() {
      this.showModal = false;
      this.selectedLesson = null;
      this.studentName = ''; // 清空輸入的名字
      // 恢復背景滾動
      document.body.style.overflow = 'auto';
    },
    getLessonDescription(type) {
      const descriptions = {
        'Hip Hop': 'Hip Hop是一種源自美國的街舞文化，強調個人風格和自由表達，結合了力量、節奏感和創意。',
        'Jazz': 'Jazz舞蹈充滿活力和表現力，結合了古典技巧與現代元素，強調身體的延展性和音樂性。',
        'K-POP': 'K-POP舞蹈融合了多種舞蹈風格，動作精準、整齊劃一，充滿活力和時尚感。',
        'Locking': 'Locking是一種節奏感強烈的街舞，以突然的停頓和鎖定動作為特色，充滿幽默感。',
        'Waacking': 'Waacking強調手臂的快速揮舞和優雅的身體線條，展現個人魅力和表演性。',
        'Breaking': 'Breaking包含了power moves、toprock、downrock等元素，是街舞文化的重要組成部分。',
        'Popping': 'Popping通過肌肉的收縮和放鬆創造機械式的動作效果，強調身體的控制力。',
        'Heels': 'Heels舞蹈穿著高跟鞋表演，強調女性魅力、優雅和自信的表達。',
        'Contemporary': 'Contemporary現代舞結合了芭蕾、現代舞和爵士舞的技巧，注重情感表達和創意。',
        'Vogue': 'Vogue起源於紐約舞廳文化，以時尚雜誌的pose和優雅的手部動作為特色。'
      };
      return descriptions[type] || '這是一個精彩的舞蹈課程，歡迎您來體驗！';
    },
    async enrollLesson(lesson) {
      // 檢查是否有輸入名字
      if (!this.studentName.trim()) {
        alert('請輸入您的姓名！');
        return;
      }

      try {
        // 準備要發送的資料
        const enrollmentData = {
          studentName: this.studentName,
          lesson: lesson
        };

        console.log('=== 發送報名資料到後端 ===');
        console.log('報名資料:', enrollmentData);

        // 發送 POST 請求到後端 (使用 Docker 內部網路)
        const response = await fetch('http://localhost:8001/api/enrollment', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(enrollmentData)
        });

        const result = await response.json();

        if (response.ok && result.success) {
          console.log('=== 後端回應成功 ===');
          console.log('報名編號:', result.enrollment.id);
          console.log('完整回應:', result);
          console.log('==================');
          
          // 顯示報名成功訊息
          alert(`${this.studentName} 您好！\n您已成功報名 ${lesson.name} 課程！\n報名編號：${result.enrollment.id}\n時間：${lesson.day} ${lesson.time}\n老師：${lesson.teacher}\n\n報名資料已儲存至資料庫！`);
          
          // 報名成功後關閉 Modal
          this.closeModal();
        } else {
          console.error('報名失敗:', result);
          alert(`報名失敗：${result.error || result.message || '未知錯誤'}`);
        }

      } catch (error) {
        console.error('=== 報名請求錯誤 ===');
        console.error('錯誤詳情:', error);
        console.log('==================');
        alert('網路錯誤，請檢查後端服務是否正常運行，或稍後再試！\n\n提示：請確保 Docker 容器正在運行。');
      }
    }
  }
};
</script>

<style src="../assets/css/Schedule.css"></style>
