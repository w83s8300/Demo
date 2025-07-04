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
            <div v-for="(lesson, lessonIndex) in getLessonsForDay(day)" :key="lessonIndex" :class="getLessonClass(lesson.type)" class="lesson-card-mobile">
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
  </div>
</template>

<script>
export default {
  name: 'Schedule',
  data() {
    return {
      danceTypes: ['全部', 'Hip Hop', 'Jazz', 'K-POP', 'Locking', 'Waacking', 'Breaking', 'Popping', 'Heels', 'Contemporary', 'Vogue'],
      selectedDanceType: '全部',
      timeSlots: [
        '09:00 - 10:00', '10:00 - 11:00', '11:00 - 12:00',
        '14:00 - 15:00', '15:00 - 16:00', '18:00 - 19:00',
        '19:00 - 20:00', '20:00 - 21:00', '21:00 - 22:00'
      ],
      daysOfWeek: ['週一', '週二', '週三', '週四', '週五', '週六', '週日'],
      classes: [
        // Monday
        { time: '10:00 - 11:00', day: '週一', name: 'Contemporary', teacher: '老師A', level: '初', type: 'Contemporary' },
        { time: '14:00 - 15:00', day: '週一', name: 'Hip Hop', teacher: '老師B', level: '中', type: 'Hip Hop' },
        { time: '19:00 - 20:00', day: '週一', name: 'Locking', teacher: '老師C', level: '進階', type: 'Locking' },
        { time: '20:00 - 21:00', day: '週一', name: 'Breaking', teacher: '老師D', level: '中', type: 'Breaking' },
        // Tuesday
        { time: '11:00 - 12:00', day: '週二', name: 'Popping', teacher: '老師E', level: '初', type: 'Popping' },
        { time: '15:00 - 16:00', day: '週二', name: 'Waacking', teacher: '老師F', level: '中', type: 'Waacking' },
        { time: '18:00 - 19:00', day: '週二', name: 'Hip Hop', teacher: '老師G', level: '進階', type: 'Hip Hop' },
        { time: '20:00 - 21:00', day: '週二', name: 'Popping', teacher: '老師H', level: '進階', type: 'Popping' },
        { time: '21:00 - 22:00', day: '週二', name: 'Vogue', teacher: '老師I', level: '進階', type: 'Vogue' },
        // Wednesday
        { time: '09:00 - 10:00', day: '週三', name: 'Jazz', teacher: '老師J', level: '入門', type: 'Jazz' },
        { time: '14:00 - 15:00', day: '週三', name: 'Heels', teacher: '老師K', level: '初', type: 'Heels' },
        { time: '19:00 - 20:00', day: '週三', name: 'Hip Hop', teacher: '老師L', level: '專攻', type: 'Hip Hop' },
        { time: '21:00 - 22:00', day: '週三', name: 'Locking', teacher: '老師M', level: '專攻', type: 'Locking' },
        // Thursday
        { time: '10:00 - 11:00', day: '週四', name: 'K-POP', teacher: '老師N', level: '入門', type: 'K-POP' },
        { time: '15:00 - 16:00', day: '週四', name: 'Vogue', teacher: '老師O', level: '中', type: 'Vogue' },
        { time: '18:00 - 19:00', day: '週四', name: 'Jazz', teacher: '老師P', level: '進階', type: 'Jazz' },
        { time: '20:00 - 21:00', day: '週四', name: 'Heels', teacher: '老師Q', level: '進階', type: 'Heels' },
        // Friday
        { time: '11:00 - 12:00', day: '週五', name: 'Breaking', teacher: '老師R', level: '入門', type: 'Breaking' },
        { time: '19:00 - 20:00', day: '週五', name: 'Waacking', teacher: '老師S', level: '進階', type: 'Waacking' },
        { time: '21:00 - 22:00', day: '週五', name: 'Popping', teacher: '老師T', level: '專攻', type: 'Popping' },
        // Saturday
        { time: '09:00 - 10:00', day: '週六', name: 'Hip Hop', teacher: '老師U', level: '初', type: 'Hip Hop' },
        { time: '14:00 - 15:00', day: '週六', name: 'Jazz', teacher: '老師V', level: '中', type: 'Jazz' },
        { time: '18:00 - 19:00', day: '週六', name: 'K-POP', teacher: '老師W', level: '進階', type: 'K-POP' },
        { time: '19:00 - 20:00', day: '週六', name: 'Street Jazz', teacher: '老師X', level: '進階', type: 'Jazz' },
        { time: '20:00 - 21:00', day: '週六', name: 'Contemporary', teacher: '老師Y', level: '進階', type: 'Contemporary' },
        // Sunday
        { time: '10:00 - 11:00', day: '週日', name: 'Locking', teacher: '老師Z', level: '初', type: 'Locking' },
        { time: '14:00 - 15:00', day: '週日', name: 'K-POP', teacher: '老師AA', level: '中', type: 'K-POP' },
        { time: '19:00 - 20:00', day: '週日', name: 'K-POP', teacher: '老師BB', level: '專攻', type: 'K-POP' },
      ]
    };
  },
  methods: {
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
    }
  }
};
</script>

<style src="../assets/css/Schedule.css"></style>
