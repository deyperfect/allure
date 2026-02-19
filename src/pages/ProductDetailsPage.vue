<template>
  <div class="service-page">

    <div class="container py-5">

      <!-- Loading -->
      <div v-if="loading" class="loading-state">
        <div class="spinner"></div>
      </div>

      <!-- Not Found -->
      <div v-else-if="!service" class="empty-state">
        <i class="bi bi-exclamation-circle fs-1"></i>
        <h4 class="mt-3">Service not found.</h4>
        <button class="btn-back mt-3" @click="goBack">
          Back to Services
        </button>
      </div>

      <!-- Service Details -->
      <div v-else>

        <!-- Back Button -->
        <button class="btn-back mb-4" @click="goBack">
          <i class="bi bi-arrow-left me-2"></i>
          Back to Services
        </button>

        <div class="row g-4">

          <!-- LEFT COLUMN -->
          <div class="col-lg-8">
            <div class="detail-card">

              <span class="service-category">
                {{ service.serviceTypeId?.category }}
              </span>

              <h1 class="service-title mt-1">
                {{ service.serviceTypeId?.name }}
              </h1>

              <p class="service-type-desc">
                {{ service.serviceTypeId?.description }}
              </p>

              <hr class="divider" />

              <div v-if="service.customDescription" class="mb-4">
                <h6 class="section-label">About this offering</h6>
                <p class="service-desc">
                  "{{ service.customDescription }}"
                </p>
              </div>

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

          <!-- RIGHT COLUMN -->
          <div class="col-lg-4">

            <!-- Provider Card -->
            <div class="provider-card mb-4">

              <h6 class="section-label mb-3">Offered by</h6>

              <div class="d-flex align-items-center gap-3 mb-3">
                <img
                  :src="service.providerId?.profileImage || 'https://placehold.co/64x64?text=?'"
                  class="provider-avatar"
                />
                <div>
                  <p class="provider-name mb-0">
                    {{ service.providerId?.firstName }}
                    {{ service.providerId?.lastName }}
                  </p>
                  <p class="provider-sub mb-0">
                    {{ service.providerId?.age }} yrs ·
                    {{ service.providerId?.gender }}
                  </p>
                </div>
              </div>

              <router-link
                :to="`/provider/${service.providerId?._id}`"
                class="btn-view-profile"
              >
                <i class="bi bi-person me-2"></i>
                View Full Profile & All Services
                <i class="bi bi-arrow-right ms-auto"></i>
              </router-link>

            </div>

            <!-- Cart Card -->
            <div class="cart-card">

              <div class="d-flex justify-content-between align-items-center mb-4">
                <span class="cart-price">${{ service.price }}</span>
                <span v-if="service.duration" class="cart-duration">
                  {{ service.duration }} min
                </span>
              </div>

              <!-- Quantity -->
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
                <span class="subtotal-value">
                  ${{ subtotal.toFixed(2) }}
                </span>
              </div>

              <!-- Add to Cart -->
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
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { Notyf } from "notyf";
import api from "../api";
import { useGlobalStore } from "../stores/global";

const route = useRoute();
const router = useRouter();
const notyf = new Notyf();
const globalStore = useGlobalStore();

const service = ref(null);
const loading = ref(true);
const addingToCart = ref(false);
const quantity = ref(1);

const goBack = () => {
  if (window.history.length > 1) {
    router.back();
  } else {
    router.push("/products");
  }
};

const subtotal = computed(() => {
  return (service.value?.price || 0) * quantity.value;
});

const incrementQty = () => quantity.value++;
const decrementQty = () => {
  if (quantity.value > 1) quantity.value--;
};

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

const addToCart = async () => {
  addingToCart.value = true;
  try {
    await api.post("/cart/add-to-cart", {
      productId: service.value._id,
      price: service.value.price,
      quantity: quantity.value,
      subtotal: subtotal.value
    });

    notyf.success(
      `Added ${quantity.value} item${quantity.value > 1 ? "s" : ""} to cart!`
    );

    globalStore.fetchCartCount();

  } catch (err) {
    notyf.error(err.response?.data?.message || "Failed to add to cart");
  } finally {
    addingToCart.value = false;
  }
};

onMounted(fetchService);
</script>

<style scoped>

/* Page */
.service-page {
  min-height: 100vh;
  background: var(--bg);
  padding-bottom: 60px;
}

/* Loading */
.loading-state {
  text-align: center;
  padding: 60px 0;
}

.spinner {
  width: 28px;
  height: 28px;
  border: 3px solid var(--border);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

.empty-state {
  text-align: center;
  padding: 60px 0;
  color: var(--text-secondary);
}

/* Back */
.btn-back {
  background: none;
  border: none;
  color: var(--text-secondary);
  font-weight: 500;
  cursor: pointer;
}

.btn-back:hover {
  color: var(--text-primary);
}

/* Cards */
.detail-card,
.provider-card,
.cart-card {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 24px;
}

/* Titles */
.service-title {
  font-size: 28px;
  font-weight: 700;
  color: var(--text-primary);
}

.service-type-desc,
.provider-sub,
.cart-note {
  color: var(--text-secondary);
}

.service-category {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  color: var(--accent);
}

.section-label {
  font-size: 11px;
  text-transform: uppercase;
  color: var(--text-muted);
}

.detail-label {
  font-size: 11px;
  color: var(--text-muted);
}

.detail-value,
.provider-name,
.cart-price,
.subtotal-value {
  font-weight: 700;
  color: var(--text-primary);
}

/* Buttons */
.btn-add-cart {
  width: 100%;
  padding: 12px;
  border-radius: 10px;
  border: none;
  background: var(--accent);
  color: white;
  font-weight: 600;
  cursor: pointer;
}

.btn-add-cart:hover:not(:disabled) {
  background: var(--accent-hover);
}

.btn-add-cart:disabled {
  opacity: 0.6;
}

/* Quantity */
.qty-btn {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 8px;
  width: 32px;
  height: 32px;
}

.qty-btn:hover:not(:disabled) {
  background: var(--accent);
  color: white;
}

.qty-value {
  font-weight: 700;
  color: var(--text-primary);
}

.subtotal-row {
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
  padding: 12px 0;
}

</style>
