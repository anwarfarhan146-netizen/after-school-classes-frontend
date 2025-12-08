<template>
  <div class="row">
    <div 
      v-for="lesson in lessons" 
      :key="lesson._id" 
      class="col-xl-4 col-lg-6 col-md-6 mb-4"
    >
      <div class="card lesson-card h-100">
        <div class="card-body d-flex flex-column">
          <!-- Subject and Icon -->
          <div class="text-center mb-3">
            <i :class="lesson.icon" class="lesson-icon"></i>
            <h5 class="card-title mb-2">{{ lesson.subject }}</h5>
            <span class="badge bg-primary subject-badge">{{ lesson.subject }}</span>
          </div>

          <!-- Description -->
          <p class="card-text text-muted text-center mb-3">
            {{ lesson.description }}
          </p>

          <!-- Lesson Details -->
          <div class="lesson-details mb-3">
            <div class="row text-center">
              <div class="col-6">
                <small class="text-muted">
                  <i class="fas fa-map-marker-alt me-1"></i>
                  Location
                </small>
                <div class="fw-bold text-primary">{{ lesson.location }}</div>
              </div>
              <div class="col-6">
                <small class="text-muted">
                  <i class="fas fa-pound-sign me-1"></i>
                  Price
                </small>
                <div class="price-tag">£{{ lesson.price }}</div>
              </div>
            </div>
          </div>

          <!-- Availability -->
          <div class="availability-section text-center mt-auto">
            <div class="mb-2">
              <small class="text-muted">
                <i class="fas fa-users me-1"></i>
                Spaces Available
              </small>
              <div 
                :class="{
                  'availability-low': lesson.space <= 2 && lesson.space > 0,
                  'availability-good': lesson.space > 2,
                  'availability-full': lesson.space === 5,
                  'availability-none': lesson.space === 0
                }"
                class="fw-bold"
              >
                <span v-if="lesson.space > 0">
                  {{ lesson.space }} / 5 spaces
                </span>
                <span v-else class="text-danger">
                  <i class="fas fa-times-circle me-1"></i>
                  Fully Booked
                </span>
              </div>
            </div>

            <!-- Progress Bar -->
            <div class="progress mb-3" style="height: 8px;">
              <div 
                class="progress-bar" 
                :class="{
                  'bg-danger': lesson.space <= 2 && lesson.space > 0,
                  'bg-warning': lesson.space > 2 && lesson.space < 5,
                  'bg-success': lesson.space === 5,
                  'bg-secondary': lesson.space === 0
                }"
                :style="{ width: (lesson.space / 5) * 100 + '%' }"
              ></div>
            </div>

            <!-- Add to Cart Button -->
            <button 
              class="btn w-100"
              :class="{
                'btn-primary': lesson.space > 0,
                'btn-secondary': lesson.space === 0
              }"
              :disabled="lesson.space === 0"
              @click="$emit('add-to-cart', lesson)"
            >
              <i class="fas fa-cart-plus me-2"></i>
              <span v-if="lesson.space > 0">Add to Cart</span>
              <span v-else>No Spaces Available</span>
            </button>

            <!-- Quick feedback when added to cart -->
            <div v-if="lesson.space < 5" class="mt-2">
              <small class="text-success">
                <i class="fas fa-check-circle me-1"></i>
                {{ 5 - lesson.space }} booked
              </small>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LessonList',
  props: {
    lessons: {
      type: Array,
      required: true
    }
  },
  emits: ['add-to-cart']
}
</script>

<style scoped>
.lesson-details {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 15px;
}

.availability-section {
  border-top: 1px solid #e9ecef;
  padding-top: 15px;
}

.progress {
  background-color: #e9ecef;
  border-radius: 4px;
}

.card-title {
  color: #2c3e50;
  font-weight: 600;
}

.subject-badge {
  font-size: 0.8em;
  padding: 0.35em 0.65em;
  background-color: #dc3545 !important;
}

.availability-none {
  color: #dc3545;
}

/* Red color overrides */
.lesson-icon {
  font-size: 2rem;
  color: #dc3545;
  margin-bottom: 1rem;
}

.price-tag {
  font-size: 1.3rem;
  font-weight: bold;
  color: #dc3545;
}

.btn-primary {
  background-color: #dc3545;
  border-color: #dc3545;
}

.btn-primary:hover:not(:disabled) {
  background-color: #c82333;
  border-color: #bd2130;
}

.btn-secondary:disabled {
  background-color: #6c757d;
  border-color: #6c757d;
}

.text-primary {
  color: #dc3545 !important;
}

/* Button hover effects */
.btn:not(:disabled):hover {
  transform: translateY(-1px);
}

.btn:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

/* Smooth transitions */
.lesson-card, .btn, .progress-bar {
  transition: all 0.3s ease;
}
</style>
