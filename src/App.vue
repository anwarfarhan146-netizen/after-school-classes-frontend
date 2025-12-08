<template>
  <div id="app">
    <!-- Navigation Header -->
    <nav class="navbar navbar-dark bg-primary">
      <div class="container">
        <a class="navbar-brand" href="#">
          <i class="fas fa-graduation-cap me-2"></i>
          After School Classes
        </a>
        <div class="navbar-text d-flex align-items-center">
          <span class="me-3" v-if="!showCartPage">
            {{ filteredLessons.length }} classes available
            <span v-if="searchTerm" class="search-results-badge">
              ({{ searchResultsCount }} results for "{{ searchTerm }}")
            </span>
          </span>
          <span class="me-3" v-else>
            Shopping Cart ({{ totalItemsInCart }} items)
          </span>

          <button 
            class="btn btn-outline-light btn-sm position-relative ms-2"
            @click="toggleCartPage"
            :disabled="!showCartButton"
          >
            <i class="fas fa-shopping-cart me-1"></i>
            Cart
            <span 
              v-if="totalItemsInCart > 0"
              class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger"
            >
              {{ totalItemsInCart }}
            </span>
          </button>
        </div>
      </div>
    </nav>

    <div class="container mt-4">
      <!-- Lessons Page -->
      <div v-if="!showCartPage">
        <div class="row">
          <div class="col-12">
            <h1 class="text-center mb-4">Available After School Classes</h1>
            <p class="text-center text-muted mb-5">
              Discover our wide range of after school activities and classes for students
            </p>
          </div>
        </div>

        <!-- Search and Sort Controls -->
        <div class="search-sort-controls mb-4">
          <div class="card">
            <div class="card-body">
              <div class="row">
                <div class="col-md-6 mb-3 mb-md-0">
                  <label class="form-label fw-bold">
                    <i class="fas fa-search me-1"></i>
                    Search Classes:
                  </label>
                  <div class="input-group">
                    <input 
                      type="text" 
                      class="form-control search-input" 
                      placeholder="Search by subject, location, price, or availability..."
                      v-model="searchTerm"
                      @input="handleSearch"
                    >
                    <button 
                      class="btn btn-outline-secondary search-clear-btn" 
                      type="button"
                      @click="clearSearch"
                      v-if="searchTerm"
                    >
                      <i class="fas fa-times"></i>
                    </button>
                  </div>
                  <div class="form-text">
                    Search as you type - results update instantly
                  </div>
                </div>

                <div class="col-md-6">
                  <div class="row">
                    <div class="col-12 mb-2">
                      <label class="form-label fw-bold">
                        <i class="fas fa-sort me-1"></i>
                        Sort Lessons By:
                      </label>
                      <select class="form-select" v-model="sortBy" @change="sortLessons">
                        <option value="subject">Subject</option>
                        <option value="location">Location</option>
                        <option value="price">Price</option>
                        <option value="space">Availability</option>
                      </select>
                    </div>
                    <div class="col-12">
                      <label class="form-label fw-bold">
                        <i class="fas fa-arrow-up-arrow-down me-1"></i>
                        Sort Order:
                      </label>
                      <div class="btn-group w-100" role="group">
                        <input 
                          type="radio" 
                          class="btn-check" 
                          name="sortOrder" 
                          id="asc" 
                          value="asc" 
                          v-model="sortOrder"
                          @change="sortLessons"
                        >
                        <label class="btn btn-outline-primary" for="asc">
                          <i class="fas fa-arrow-up-a-z me-1"></i>
                          Ascending
                        </label>

                        <input 
                          type="radio" 
                          class="btn-check" 
                          name="sortOrder" 
                          id="desc" 
                          value="desc" 
                          v-model="sortOrder"
                          @change="sortLessons"
                        >
                        <label class="btn btn-outline-primary" for="desc">
                          <i class="fas fa-arrow-down-z-a me-1"></i>
                          Descending
                        </label>
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <div class="row mt-3">
                <div class="col-12">
                  <div class="alert alert-info py-2 mb-0" v-if="sortBy || searchTerm">
                    <small>
                      <i class="fas fa-info-circle me-1"></i>
                      <span v-if="searchTerm">
                        Showing {{ searchResultsCount }} results for "<strong>{{ searchTerm }}</strong>"
                        <span v-if="sortBy"> • </span>
                      </span>
                      <span v-if="sortBy">
                        Sorted by <strong>{{ getSortDisplayName(sortBy) }}</strong> in 
                        <strong>{{ sortOrder === 'asc' ? 'Ascending' : 'Descending' }}</strong> order
                      </span>
                    </small>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div v-if="searchTerm && filteredLessons.length === 0" class="text-center py-5 no-results-animation">
          <i class="fas fa-search fa-4x text-muted mb-3"></i>
          <h3 class="text-muted">No classes found</h3>
          <p class="text-muted mb-4">
            No results found for "<strong>{{ searchTerm }}</strong>". Try adjusting your search terms.
          </p>
          <button class="btn btn-primary" @click="clearSearch">
            <i class="fas fa-times me-2"></i>
            Clear Search
          </button>
        </div>

        <div v-if="totalItemsInCart > 0 && (!searchTerm || filteredLessons.length > 0)" class="alert alert-success mb-4">
          <div class="d-flex justify-content-between align-items-center">
            <div>
              <i class="fas fa-shopping-cart me-2"></i>
              <strong>{{ totalItemsInCart }}</strong> item(s) in cart • 
              <strong>£{{ totalCartPrice }}</strong> total
            </div>
            <button class="btn btn-sm btn-outline-success" @click="showCartPage = true">
              View Cart
            </button>
          </div>
        </div>

        <LessonList 
          :lessons="filteredLessons" 
          @add-to-cart="addToCart"
          v-if="!searchTerm || filteredLessons.length > 0"
        />

        <div class="row mt-5" v-if="!searchTerm || filteredLessons.length > 0">
          <div class="col-12">
            <div class="card bg-light">
              <div class="card-body text-center">
                <div class="row">
                  <div class="col-md-3">
                    <h4>{{ filteredLessons.length }}</h4>
                    <p class="text-muted mb-0">
                      {{ searchTerm ? 'Filtered' : 'Total' }} Classes
                    </p>
                  </div>
                  <div class="col-md-3">
                    <h4>{{ totalSpaces }}</h4>
                    <p class="text-muted mb-0">Total Spaces</p>
                  </div>
                  <div class="col-md-3">
                    <h4>{{ availableSpaces }}</h4>
                    <p class="text-muted mb-0">Available Spaces</p>
                  </div>
                  <div class="col-md-3">
                    <h4>{{ totalItemsInCart }}</h4>
                    <p class="text-muted mb-0">In Cart</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Shopping Cart Page -->
      <div v-else>
        <div class="row mb-4">
          <div class="col-12">
            <div class="d-flex justify-content-between align-items-center">
              <h1 class="mb-0">
                <i class="fas fa-shopping-cart me-2"></i>
                Shopping Cart
              </h1>
              <button class="btn btn-outline-primary" @click="toggleCartPage">
                <i class="fas fa-arrow-left me-2"></i>
                Back to Lessons
              </button>
            </div>
            <p class="text-muted">Review your selected classes and manage your cart</p>
          </div>
        </div>

        <div class="row">
          <div class="col-12">
            <div v-if="cart.length === 0" class="text-center py-5">
              <i class="fas fa-shopping-cart empty-cart-icon text-muted mb-3"></i>
              <h3 class="text-muted">Your cart is empty</h3>
              <p class="text-muted mb-4">Add some classes to get started!</p>
              <button class="btn btn-primary" @click="toggleCartPage">
                <i class="fas fa-book me-2"></i>
                Browse Classes
              </button>
            </div>

            <div v-else>
              <div class="card mb-4">
                <div class="card-header bg-light">
                  <h5 class="mb-0">
                    <i class="fas fa-list me-2"></i>
                    Cart Items ({{ totalItemsInCart }})
                  </h5>
                </div>
                <div class="card-body p-0">
                  <div 
                    v-for="item in cart" 
                    :key="item.id"
                    class="cart-item border-bottom p-4"
                  >
                    <div class="row align-items-center">
                      <div class="col-md-6">
                        <div class="d-flex align-items-center">
                          <i :class="item.icon" class="fa-2x text-primary me-3"></i>
                          <div>
                            <h5 class="mb-1">{{ item.subject }}</h5>
                            <p class="text-muted mb-1">
                              <i class="fas fa-map-marker-alt me-1"></i>
                              {{ item.location }}
                            </p>
                            <p class="text-success mb-0 fw-bold">
                              £{{ item.price }} per space
                            </p>
                          </div>
                        </div>
                      </div>

                      <div class="col-md-3">
                        <div class="d-flex align-items-center">
                          <label class="form-label me-3 mb-0">Quantity:</label>
                          <div class="input-group" style="width: 120px;">
                            <button 
                              class="btn btn-outline-secondary"
                              type="button"
                              @click="decreaseQuantity(item)"
                              :disabled="item.quantity <= 1"
                            >
                              <i class="fas fa-minus"></i>
                            </button>
                            <input 
                              type="text" 
                              class="form-control text-center"
                              :value="item.quantity"
                              readonly
                            >
                            <button 
                              class="btn btn-outline-secondary"
                              type="button"
                              @click="increaseQuantity(item)"
                            >
                              <i class="fas fa-plus"></i>
                            </button>
                          </div>
                        </div>
                      </div>

                      <div class="col-md-3">
                        <div class="d-flex justify-content-between align-items-center">
                          <div class="text-end">
                            <div class="h5 mb-0 text-primary">
                              £{{ (item.price * item.quantity).toFixed(2) }}
                            </div>
                            <small class="text-muted">
                              £{{ item.price }} × {{ item.quantity }}
                            </small>
                          </div>
                          <button 
                            class="btn btn-outline-danger ms-3"
                            @click="removeFromCart(item)"
                            title="Remove from cart"
                          >
                            <i class="fas fa-trash"></i>
                          </button>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div class="card-footer">
                  <div class="row align-items-center">
                    <div class="col-md-6">
                      <div class="d-flex align-items-center">
                        <i class="fas fa-info-circle text-primary me-2"></i>
                        <small class="text-muted">
                          Items in cart will be held for 30 minutes
                        </small>
                      </div>
                    </div>
                    <div class="col-md-6 text-end">
                      <div class="h4 mb-2">
                        Total: <span class="text-success">£{{ totalCartPrice.toFixed(2) }}</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <div class="card">
                <div class="card-header bg-light">
                  <h5 class="mb-0">
                    <i class="fas fa-credit-card me-2"></i>
                    Checkout
                  </h5>
                </div>
                <div class="card-body">
                  <form @submit.prevent="submitOrder">
                    <div class="row">
                      <div class="col-md-6">
                        <h6 class="mb-3">Customer Information</h6>

                        <div class="mb-3">
                          <label for="customerName" class="form-label">
                            Full Name <span class="text-danger">*</span>
                          </label>
                          <input 
                            type="text" 
                            class="form-control" 
                            id="customerName"
                            v-model="checkoutForm.name"
                            :class="{ 
                              'is-invalid': checkoutForm.errors.name,
                              'is-valid': checkoutForm.name && !checkoutForm.errors.name 
                            }"
                            placeholder="Enter your full name"
                            @input="validateName"
                          >
                          <div v-if="checkoutForm.errors.name" class="invalid-feedback">
                            {{ checkoutForm.errors.name }}
                          </div>
                          <div v-else class="form-text">
                            Letters and spaces only
                          </div>
                        </div>

                        <div class="mb-3">
                          <label for="customerPhone" class="form-label">
                            Phone Number <span class="text-danger">*</span>
                          </label>
                          <input 
                            type="tel" 
                            class="form-control" 
                            id="customerPhone"
                            v-model="checkoutForm.phone"
                            :class="{ 
                              'is-invalid': checkoutForm.errors.phone,
                              'is-valid': checkoutForm.phone && !checkoutForm.errors.phone 
                            }"
                            placeholder="Enter your phone number"
                            @input="validatePhone"
                          >
                          <div v-if="checkoutForm.errors.phone" class="invalid-feedback">
                            {{ checkoutForm.errors.phone }}
                          </div>
                          <div v-else class="form-text">
                            Numbers only (no spaces or special characters)
                          </div>
                        </div>
                      </div>

                      <div class="col-md-6">
                        <h6 class="mb-3">Order Summary</h6>
                        <div class="order-summary">
                          <div v-for="item in cart" :key="item.id" class="d-flex justify-content-between mb-2">
                            <div>
                              <strong>{{ item.subject }}</strong>
                              <br>
                              <small class="text-muted">
                                {{ item.quantity }} × £{{ item.price }} 
                                • {{ item.location }}
                              </small>
                            </div>
                            <div class="text-end">
                              <div>£{{ (item.price * item.quantity).toFixed(2) }}</div>
                            </div>
                          </div>
                          <hr>
                          <div class="d-flex justify-content-between fw-bold fs-5">
                            <span>Total:</span>
                            <span class="text-success">£{{ totalCartPrice.toFixed(2) }}</span>
                          </div>
                        </div>
                      </div>
                    </div>

                    <div class="row mt-4">
                      <div class="col-12">
                        <div class="d-grid">
                          <button 
                            type="submit" 
                            class="btn btn-success btn-lg"
                            :disabled="!isCheckoutValid || checkoutForm.submitting"
                          >
                            <span v-if="checkoutForm.submitting" class="spinner-border spinner-border-sm me-2"></span>
                            <i v-else class="fas fa-paper-plane me-2"></i>
                            {{ checkoutForm.submitting ? 'Processing...' : 'Complete Order' }}
                          </button>
                        </div>
                      </div>
                    </div>
                  </form>
                </div>
              </div>

              <div v-if="checkoutForm.showConfirmation" class="modal fade show d-block" tabindex="-1" style="background: rgba(0,0,0,0.5)">
                <div class="modal-dialog modal-dialog-centered">
                  <div class="modal-content">
                    <div class="modal-header bg-success text-white">
                      <h5 class="modal-title">
                        <i class="fas fa-check-circle me-2"></i>
                        Order Confirmed!
                      </h5>
                    </div>
                    <div class="modal-body text-center">
                      <i class="fas fa-check-circle fa-4x text-success mb-3"></i>
                      <h4>Thank you for your order!</h4>
                      <p class="mb-1"><strong>Order Reference:</strong> #{{ checkoutForm.orderReference }}</p>
                      <p class="mb-1"><strong>Customer:</strong> {{ checkoutForm.name }}</p>
                      <p class="mb-3"><strong>Total:</strong> £{{ totalCartPrice.toFixed(2) }}</p>
                      <p class="text-muted">
                        A confirmation email has been sent to your registered phone number.
                        Our team will contact you shortly to complete the registration process.
                      </p>
                    </div>
                    <div class="modal-footer">
                      <button type="button" class="btn btn-primary" @click="confirmOrder">
                        Continue Shopping
                      </button>
                    </div>
                  </div>
                </div>
              </div>

              <div class="row mt-3">
                <div class="col-12">
                  <div class="d-flex gap-2 justify-content-end">
                    <button class="btn btn-outline-secondary" @click="clearCart">
                      <i class="fas fa-trash me-2"></i>
                      Clear Cart
                    </button>
                    <button class="btn btn-outline-primary" @click="toggleCartPage">
                      <i class="fas fa-plus me-2"></i>
                      Add More Classes
                    </button>
                  </div>
                </div>
              </div>

            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

// Main Vue App component for CST3144 Coursework
<script>
import LessonList from './components/LessonList.vue';

// API base taken from env var at build time; fall back to localhost for dev
const API_BASE_URL = process.env.VUE_APP_API_BASE || 'http://localhost:3000';

export default {
  name: 'App',
  components: { LessonList },
  data() {
    
    return {
      sortBy: 'subject',
      sortOrder: 'asc',
      showCartPage: false,
      cart: [],
      searchTerm: '',
      searchTimeout: null,
      checkoutForm: {
        name: '',
        phone: '',
        errors: { name: '', phone: '' },
        submitting: false,
        showConfirmation: false,
        orderReference: ''
      },
      lessons: []
    };
  },
  computed: {
    searchedLessons() {
      if (!this.searchTerm) return this.lessons;
      const searchTerm = this.searchTerm.toLowerCase();
      return this.lessons.filter(lesson =>
        (lesson.subject && lesson.subject.toLowerCase().includes(searchTerm)) ||
        (lesson.location && lesson.location.toLowerCase().includes(searchTerm)) ||
        (lesson.price && lesson.price.toString().includes(searchTerm)) ||
        (lesson.space && lesson.space.toString().includes(searchTerm)) ||
        (lesson.description && lesson.description.toLowerCase().includes(searchTerm))
      );
    },
    filteredLessons() {
      let sorted = [...this.searchedLessons];
      sorted.sort((a, b) => {
        let aValue = a[this.sortBy];
        let bValue = b[this.sortBy];
        if (typeof aValue === 'string') aValue = aValue.toLowerCase();
        if (typeof bValue === 'string') bValue = bValue.toLowerCase();
        if (this.sortOrder === 'asc') return aValue > bValue ? 1 : -1;
        return aValue < bValue ? 1 : -1;
      });
      return sorted;
    },
    totalSpaces() { return this.filteredLessons.length * 5; },
    availableSpaces() { return this.filteredLessons.reduce((t, l) => t + (l.space || 0), 0); },
    totalItemsInCart() { return this.cart.reduce((t, i) => t + i.quantity, 0); },
    totalCartPrice() { return this.cart.reduce((t, i) => t + (i.price * i.quantity), 0); },
    showCartButton() { return this.totalItemsInCart > 0; },
    isCheckoutValid() {
      return this.checkoutForm.name &&
             this.checkoutForm.phone &&
             !this.checkoutForm.errors.name &&
             !this.checkoutForm.errors.phone;
    },
    searchResultsCount() { return this.filteredLessons.length; }
  },
  methods: {
    sortLessons() {},
    getSortDisplayName(sortBy) {
      const displayNames = { subject: 'Subject', location: 'Location', price: 'Price', space: 'Availability' };
      return displayNames[sortBy] || sortBy;
    },
    handleSearch() {
      clearTimeout(this.searchTimeout);
      this.searchTimeout = setTimeout(() => {
        console.log(`Searching for: "${this.searchTerm}" - Found ${this.searchResultsCount} results`);
      }, 300);
    },
    clearSearch() { this.searchTerm = ''; },

    async fetchLessons() {
      try {
        const res = await fetch(`${API_BASE_URL}/lessons`);
        if (!res.ok) throw new Error('Failed to fetch lessons');
        const data = await res.json();
        // ensure numeric fields parsed
        this.lessons = data.map(l => ({ ...l, price: Number(l.price), space: Number(l.space) }));
      } catch (err) {
        console.error('Error fetching lessons:', err);
        this.lessons = [];
      }
    },

    addToCart(lesson) {
      if (lesson.space > 0) {
        const existing = this.cart.find(i => i.id === lesson._id);
        if (existing) existing.quantity += 1;
        else this.cart.push({
          id: lesson._id,
          subject: lesson.subject,
          location: lesson.location,
          price: lesson.price,
          icon: lesson.icon,
          quantity: 1
        });
        lesson.space -= 1;
      }
    },

    toggleCartPage() { this.showCartPage = !this.showCartPage; },

    increaseQuantity(item) {
      const lesson = this.lessons.find(l => l._id === item.id);
      if (lesson && lesson.space > 0) {
        item.quantity += 1;
        lesson.space -= 1;
      }
    },

    decreaseQuantity(item) {
      if (item.quantity > 1) {
        const lesson = this.lessons.find(l => l._id === item.id);
        if (lesson) {
          item.quantity -= 1;
          lesson.space += 1;
        }
      }
    },

    removeFromCart(item) {
      const idx = this.cart.findIndex(c => c.id === item.id);
      if (idx !== -1) {
        const lesson = this.lessons.find(l => l._id === item.id);
        if (lesson) lesson.space += item.quantity;
        this.cart.splice(idx, 1);
      }
    },

    clearCart() {
      if (confirm('Are you sure you want to clear your cart? All items will be removed.')) {
        this.cart.forEach(item => {
          const lesson = this.lessons.find(l => l._id === item.id);
          if (lesson) lesson.space += item.quantity;
        });
        this.cart = [];
      }
    },

    validateName() {
      const nameRegex = /^[A-Za-z\s]+$/;
      if (!this.checkoutForm.name.trim()) this.checkoutForm.errors.name = 'Name is required';
      else if (!nameRegex.test(this.checkoutForm.name)) this.checkoutForm.errors.name = 'Name must contain only letters and spaces';
      else this.checkoutForm.errors.name = '';
    },

    validatePhone() {
      const phoneRegex = /^\d+$/;
      if (!this.checkoutForm.phone.trim()) this.checkoutForm.errors.phone = 'Phone number is required';
      else if (!phoneRegex.test(this.checkoutForm.phone)) this.checkoutForm.errors.phone = 'Phone number must contain only numbers';
      else this.checkoutForm.errors.phone = '';
    },

    async submitOrder() {
      this.validateName();
      this.validatePhone();
      if (!this.isCheckoutValid) return;
      this.checkoutForm.submitting = true;

      try {
        const orderData = {
          name: this.checkoutForm.name,
          phone: this.checkoutForm.phone,
          lessonIDs: this.cart.map(item => item.id),
          spaces: this.cart.map(item => item.quantity)
        };

        const res = await fetch(`${API_BASE_URL}/orders`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(orderData)
        });

        if (!res.ok) throw new Error('Failed to submit order');
        const responseJson = await res.json();

        // Update lesson spaces in backend
        for (const item of this.cart) {
          const lesson = this.lessons.find(l => l._id === item.id);
          if (lesson) {
            await fetch(`${API_BASE_URL}/lessons/${item.id}`, {
              method: 'PUT',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ space: lesson.space })
            });
          }
        }

        this.checkoutForm.orderReference = 'ORD' + Date.now().toString().slice(-6);
        this.checkoutForm.showConfirmation = true;

      } catch (error) {
        console.error('Error submitting order:', error);
        alert('Error submitting order. Please try again.');
      } finally {
        this.checkoutForm.submitting = false;
      }
    },

    confirmOrder() {
      this.cart = [];
      this.checkoutForm = {
        name: '',
        phone: '',
        errors: { name: '', phone: '' },
        submitting: false,
        showConfirmation: false,
        orderReference: ''
      };
      this.showCartPage = false;
      this.fetchLessons();
    }
  },
  mounted() {
    this.fetchLessons();
  }
};
</script>
// more styles
<style scoped>
.navbar-brand {
  font-size: 1.5rem;
  font-weight: bold;
}

.search-results-badge {
  background-color: #ffc107;
  color: #000;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 0.8rem;
  margin-left: 8px;
}

.btn:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

.cart-item {
  transition: background-color 0.2s ease;
}
</style>
