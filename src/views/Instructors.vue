<template>
  <div class="instructors-page">
    <!-- 1. Banner -->
    <div class="banner">
      <img src="https://picsum.photos/1920/500?random=14" alt="Instructors Banner" class="banner-img">
      <div class="banner-text">
        <h1>師資團隊 / INSTRUCTORS</h1>
      </div>
    </div>

    <!-- 2. Instructors Grid -->
    <div class="container my-5">
      <div class="row">
        <!-- Instructor 1 -->
        <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
          <div class="card instructor-card">
            <img src="https://picsum.photos/400/500?random=15" class="card-img-top" alt="Instructor 1">
            <div class="card-body">
              <h5 class="card-title">Teacher A</h5>
              <p class="card-text">Hip Hop / Locking</p>
              <a href="#" class="btn btn-outline-dark btn-sm">View Profile</a>
            </div>
          </div>
        </div>
        <!-- Instructor 2 -->
        <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
          <div class="card instructor-card">
            <img src="https://picsum.photos/400/500?random=16" class="card-img-top" alt="Instructor 2">
            <div class="card-body">
              <h5 class="card-title">Teacher B</h5>
              <p class="card-text">Popping / Breaking</p>
              <a href="#" class="btn btn-outline-dark btn-sm">View Profile</a>
            </div>
          </div>
        </div>
        <!-- Instructor 3 -->
        <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
          <div class="card instructor-card">
            <img src="https://picsum.photos/400/500?random=17" class="card-img-top" alt="Instructor 3">
            <div class="card-body">
              <h5 class="card-title">Teacher C</h5>
              <p class="card-text">Jazz / Contemporary</p>
              <a href="#" class="btn btn-outline-dark btn-sm">View Profile</a>
            </div>
          </div>
        </div>
        <!-- Instructor 4 -->
        <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
          <div class="card instructor-card">
            <img src="https://picsum.photos/400/500?random=18" class="card-img-top" alt="Instructor 4">
            <div class="card-body">
              <h5 class="card-title">Teacher D</h5>
              <p class="card-text">K-POP / MV Dance</p>
              <a href="#" class="btn btn-outline-dark btn-sm">View Profile</a>
            </div>
          </div>
        </div>
        <!-- Instructor 5 -->
        <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
          <div class="card instructor-card">
            <img src="https://picsum.photos/400/500?random=19" class="card-img-top" alt="Instructor 5">
            <div class="card-body">
              <h5 class="card-title">Teacher E</h5>
              <p class="card-text">Waacking / Vogue</p>
              <a href="#" class="btn btn-outline-dark btn-sm">View Profile</a>
            </div>
          </div>
        </div>
        <!-- Instructor 6 -->
        <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
          <div class="card instructor-card">
            <img src="https://picsum.photos/400/500?random=20" class="card-img-top" alt="Instructor 6">
            <div class="card-body">
              <h5 class="card-title">Teacher F</h5>
              <p class="card-text">Street Jazz / Heels</p>
              <a href="#" class="btn btn-outline-dark btn-sm">View Profile</a>
            </div>
          </div>
        </div>
        <!-- Add more instructors as needed -->
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Instructors'
}
</script>

<style src="../assets/css/Instructors.css"></style>