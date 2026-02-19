<template>
  <div class="container py-5">

    <div v-if="loading" class="text-center py-5">
      <div class="spinner-border text-warning"></div>
    </div>

    <div v-else-if="!service" class="text-center py-5">
      <i class="bi bi-exclamation-circle fs-1 text-muted"></i>
      <h4 class="mt-3">Service not found.</h4>
      <button class="btn btn-outline-secondary mt-3" @click="router.push('/products/active')">
        Back to Services
      </button>
    </div>

    <!-- Service Details -->
    <div v-else>

      <!-- Back -->
      <button class="btn-back mb-4" @click="router.push('/products/active')">
        <i class="bi bi-arrow-left me-2"></i>Back to Services
      </button>

      <div class="row g-4">

        <!-- ── Left: Service Info ── -->
        <div class="col-lg-8">
          <div class="detail-card">

            <!-- Category & Name -->
            <span class="service-category">{{ service.serviceTypeId?.category }}</span>
            <h1 class="service-title mt-1">{{ service.serviceTypeId?.name }}</h1>

            <!-- Type Description -->
            <p class="service-type-desc">{{ service.serviceTypeId?.description }}</p>

            <hr class="divider" />

            <!-- Custom Description -->
            <div v-if="service.customDescription" class="section mb-4">
              <h6 class="section-label">About this offering</h6>
              <p class="service-desc">"{{ service.customDescription }}"</p>
            </div>

            <!-- Details -->
            <div class="details-row">
              <div class="detail-item">
                <i class="bi bi-tag"></i>
                <div>
                  <p class="detail-label">Price</p>
                  <p class="detail-value">${{ service.price }}</p>
                </div>
              </div>
              <div v-if="service.duration" class="detail-item">
                <i class="bi bi-clock"></i>
                <div>
                  <p class="detail-label">Duration</p>
                  <p class="detail-value">{{ service.duration }} minutes</p>
                </div>
              </div>
              <div v-if="service.availabilityNotes" class="detail-item">
                <i class="bi bi-calendar3"></i>
                <div>
                  <p class="detail-label">Availability</p>
                  <p class="detail-value">{{ service.availabilityNotes }}</p>
                </div>
              </div>
            </div>

          </div>
        </div>

        <div class="col-lg-4">

          <!-- Provider Card -->
          <div class="provider-card mb-4">
            <h6 class="section-label mb-3">Offered by</h6>
            <div class="d-flex align-items-center gap-3 mb-3">
              <img
                :src="service.providerId?.profileImage || 'https://placehold.co/64x64?text=?'"
                class="provider-avatar"
                alt="Provider"
              />
              <div>
                <p class="provider-name mb-0">
                  {{ service.providerId?.firstName }} {{ service.providerId?.lastName }}
                </p>
                <p class="provider-sub mb-0">
                  {{ service.providerId?.age }} yrs · {{ service.providerId?.gender }}
                </p>
              </div>
            </div>

            <!-- View Profile Link -->
            <router-link
              :to="`/provider/${service.providerId?._id}`"
              class="btn-view-profile"
            >
              <i class="bi bi-person me-2"></i>
              View Full Profile & All Services
              <i class="bi bi-arrow-right ms-auto"></i>
            </router-link>
          </div>

          <!-- Add to Cart Card -->
          <div class="cart-card">

            <!-- Price & Duration -->
            <div class="d-flex justify-content-between align-items-center mb-4">
              <span class="cart-price">${{ service.price }}</span>
              <span v-if="service.duration" class="cart-duration">
                {{ service.duration }} min
              </span>
            </div>

            <!-- Quantity Selector -->
            <div class="qty-section mb-3">
              <p class="detail-label mb-2">Quantity</p>
              <div class="qty-controls">
                <button class="qty-btn" @click="decrementQty" :disabled="quantity <= 1">
                  <i class="bi bi-dash"></i>
                </button>
                <span class="qty-value">{{ quantity }}</span>
                <button class="qty-btn" @click="incrementQty">
                  <i class="bi bi-plus"></i>
                </button>
              </div>
            </div>

            <!-- Subtotal -->
            <div class="subtotal-row mb-4">
              <span class="subtotal-label">Subtotal</span>
              <span class="subtotal-value">${{ subtotal.toFixed(2) }}</span>
            </div>

            <button
              class="btn-add-cart"
              :disabled="addingToCart"
              @click="addToCart"
            >
              <span v-if="addingToCart" class="btn-spinner me-2"></span>
              <i v-else class="bi bi-cart-plus me-2"></i>
              {{ addingToCart ? 'Adding...' : 'Add to Cart' }}
            </button>

            <p class="cart-note mt-3">
              <i class="bi bi-shield-check me-1"></i>
              Secure checkout · Cancel anytime
            </p>
          </div>

        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { Notyf } from "notyf";
import api from "../api";
import { useGlobalStore } from '../stores/global'

const route = useRoute();
const router = useRouter();
const notyf = new Notyf();
const globalStore = useGlobalStore()

const service = ref(null);
const loading = ref(true);
const addingToCart = ref(false);
const quantity = ref(1);

// Subtotal
const subtotal = computed(() => {
  return (service.value?.price || 0) * quantity.value;
});

// Quantity Controls
const incrementQty = () => quantity.value++;
const decrementQty = () => { if (quantity.value > 1) quantity.value--; };


// Fetch Service
const fetchService = async () => {
  try {
    const { data } = await api.get(`/services/${route.params.id}`);
    service.value = data;
  } catch (error) {
    console.error("Error fetching service:", error);
  } finally {
    loading.value = false;
  }
};

// Add to Cart
const addToCart = async () => {
  addingToCart.value = true;
  try {
    const price = service.value.price;

    await api.post("/cart/add-to-cart", {
      productId: service.value._id, 
      price,
      quantity: quantity.value,
      subtotal: subtotal.value
    });
    notyf.success(`Added ${quantity.value} item${quantity.value > 1 ? 's' : ''} to cart!`);
    globalStore.fetchCartCount()
  } catch (err) {
    notyf.error(err.response?.data?.message || "Failed to add to cart");
  } finally {
    addingToCart.value = false;
  }
};

onMounted(() => {
  fetchService();
});
</script>

<style scoped>
/* ── Back Button ── */
.btn-back {
  display: inline-flex;
  align-items: center;
  background: none;
  border: none;
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  padding: 0;
  transition: color 0.2s;
}

.btn-back:hover {
  color: var(--text-primary);
}

/* ── Detail Card ── */
.detail-card {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 32px;
}

.service-category {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  color: var(--accent);
}

.service-title {
  font-size: 28px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 8px;
}

.service-type-desc {
  color: var(--text-secondary);
  font-size: 14px;
  margin-bottom: 0;
}

.divider {
  border-color: var(--border);
  margin: 24px 0;
}

.section-label {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  color: var(--text-muted);
  margin-bottom: 8px;
}

.service-desc {
  font-size: 15px;
  color: var(--text-secondary);
  font-style: italic;
  line-height: 1.7;
}

/* ── Details Row ── */
.details-row {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
}

.detail-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.detail-item i {
  font-size: 20px;
  color: var(--accent);
  margin-top: 2px;
}

.detail-label {
  font-size: 11px;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 2px;
}

.detail-value {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0;
}

/* ── Provider Card ── */
.provider-card {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 24px;
}

.provider-avatar {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid var(--accent);
  flex-shrink: 0;
}

.provider-name {
  font-size: 15px;
  font-weight: 700;
  color: var(--text-primary);
}

.provider-sub {
  font-size: 12px;
  color: var(--text-secondary);
}

.btn-view-profile {
  display: flex;
  align-items: center;
  width: 100%;
  padding: 10px 14px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--bg);
  color: var(--text-primary);
  font-size: 13px;
  font-weight: 500;
  text-decoration: none;
  transition: all 0.2s;
  gap: 4px;
}

.btn-view-profile:hover {
  background: var(--hover-bg);
  border-color: var(--accent);
  color: var(--accent);
}

/* ── Cart Card ── */
.cart-card {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 24px;
}

.cart-price {
  font-size: 28px;
  font-weight: 700;
  color: var(--text-primary);
}

.cart-duration {
  font-size: 13px;
  color: var(--text-secondary);
  background: var(--hover-bg);
  border: 1px solid var(--border);
  border-radius: 999px;
  padding: 4px 12px;
}

/* ── Quantity Controls ── */
.qty-section {
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 12px 14px;
}

.qty-controls {
  display: flex;
  align-items: center;
  gap: 0;
}

.qty-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--card-bg);
  color: var(--text-primary);
  font-size: 16px;
  cursor: pointer;
  transition: all 0.15s;
  flex-shrink: 0;
}

.qty-btn:hover:not(:disabled) {
  background: var(--accent);
  border-color: var(--accent);
  color: white;
}

.qty-btn:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.qty-value {
  min-width: 48px;
  text-align: center;
  font-size: 16px;
  font-weight: 700;
  color: var(--text-primary);
}

/* ── Subtotal Row ── */
.subtotal-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
}

.subtotal-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.subtotal-value {
  font-size: 22px;
  font-weight: 700;
  color: var(--text-primary);
}

/* ── Add to Cart Button ── */
.btn-add-cart {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 13px;
  background: var(--accent);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-add-cart:hover:not(:disabled) {
  background: var(--accent-hover);
  transform: translateY(-1px);
}

.btn-add-cart:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-spinner {
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}

@keyframes spin { to { transform: rotate(360deg); } }

.cart-note {
  font-size: 12px;
  color: var(--text-muted);
  text-align: center;
  margin-bottom: 0;
}
</style>