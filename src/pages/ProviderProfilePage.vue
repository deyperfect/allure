<template>
  <div class="container py-5">

    <!-- Loading -->
    <div v-if="loading" class="text-center py-5">
      <div class="spinner-border text-warning"></div>
    </div>

    <!-- Not Found -->
    <div v-else-if="!provider" class="text-center py-5">
      <i class="bi bi-person-x fs-1 text-muted"></i>
      <h4 class="mt-3">Provider not found.</h4>
      <button class="btn btn-outline-secondary mt-3" @click="router.push('/products')">
        Back to Services
      </button>
    </div>

    <!-- Profile -->
    <div v-else>

      <!-- Back -->
      <button class="btn-back mb-4" @click="router.go(-1)">
        <i class="bi bi-arrow-left me-2"></i>Back
      </button>

      <!-- Hero Banner -->
      <div class="profile-hero mb-4">
        <div class="hero-bg"></div>
        <div class="hero-content">
          <img
            :src="provider.profileImage || 'https://placehold.co/120x120?text=?'"
            class="profile-avatar"
            alt="Provider"
          />
          <div class="hero-info">
            <h1 class="provider-name">{{ provider.firstName }} {{ provider.lastName }}</h1>
            <div class="provider-meta">
              <span v-if="provider.age" class="meta-tag">
                <i class="bi bi-person me-1"></i>{{ provider.age }} yrs
              </span>
              <span v-if="provider.gender" class="meta-tag">
                <i class="bi bi-gender-ambiguous me-1"></i>{{ provider.gender }}
              </span>
              <span class="meta-tag">
                <i class="bi bi-briefcase me-1"></i>{{ services.length }} service{{ services.length !== 1 ? 's' : '' }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <!-- Bio -->
      <div v-if="provider.bio" class="bio-card mb-4">
        <h6 class="section-label mb-2">About</h6>
        <p class="bio-text">{{ provider.bio }}</p>
      </div>

      <!-- Services -->
      <div class="services-section">
        <h6 class="section-label mb-3">Services Offered</h6>

        <div v-if="services.length === 0" class="text-center py-5 text-muted">
          <i class="bi bi-inbox fs-2"></i>
          <p class="mt-2">No active services at the moment.</p>
        </div>

        <div v-else class="services-grid">
          <div
            v-for="svc in services"
            :key="svc._id"
            class="service-card"
            @click="router.push(`/products/${svc._id}`)"
          >
            <!-- Category badge -->
            <span class="svc-category">{{ svc.serviceTypeId?.category }}</span>
            <h5 class="svc-name">{{ svc.serviceTypeId?.name }}</h5>
            <p class="svc-desc">{{ svc.serviceTypeId?.description }}</p>

            <div v-if="svc.customDescription" class="svc-custom">
              "{{ svc.customDescription }}"
            </div>

            <div class="svc-footer">
              <div class="svc-details">
                <span v-if="svc.duration" class="svc-meta">
                  <i class="bi bi-clock me-1"></i>{{ svc.duration }} min
                </span>
                <span v-if="svc.availabilityNotes" class="svc-meta">
                  <i class="bi bi-calendar3 me-1"></i>{{ svc.availabilityNotes }}
                </span>
              </div>
              <span class="svc-price">${{ svc.price }}</span>
            </div>

            <div class="svc-cta">
              View & Book <i class="bi bi-arrow-right ms-1"></i>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import api from "../api";

const route    = useRoute();
const router   = useRouter();

const provider = ref(null);
const services = ref([]);
const loading  = ref(true);

const fetchProviderAndServices = async () => {
    console.log("Provider ID from route:", route.params.id);
  try {
    const [providerRes, servicesRes] = await Promise.all([
      api.get(`/users/${route.params.id}`),
      api.get(`/services/provider/${route.params.id}`)
    ]);
    provider.value = providerRes.data;
    services.value = servicesRes.data;
  } catch (error) {
    console.error("Error fetching provider profile:", error);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchProviderAndServices();
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
.btn-back:hover { color: var(--text-primary); }

/* ── Hero ── */
.profile-hero {
  position: relative;
  border-radius: 20px;
  overflow: hidden;
  border: 1px solid var(--border);
}

.hero-bg {
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, var(--accent) 0%, transparent 60%);
  opacity: 0.12;
}

.hero-content {
  position: relative;
  display: flex;
  align-items: center;
  gap: 24px;
  padding: 32px;
  background: var(--card-bg);
}

.profile-avatar {
  width: 96px;
  height: 96px;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid var(--accent);
  flex-shrink: 0;
}

.provider-name {
  font-size: 26px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 10px;
}

.provider-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.meta-tag {
  font-size: 12px;
  font-weight: 500;
  color: var(--text-secondary);
  background: var(--hover-bg);
  border: 1px solid var(--border);
  border-radius: 999px;
  padding: 4px 12px;
}

/* ── Bio ── */
.bio-card {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 24px;
}

.section-label {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  color: var(--text-muted);
}

.bio-text {
  font-size: 15px;
  color: var(--text-secondary);
  line-height: 1.7;
  margin: 0;
}

/* ── Services Grid ── */
.services-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
}

.service-card {
  background: var(--card-bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.service-card:hover {
  border-color: var(--accent);
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.08);
}

.svc-category {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  color: var(--accent);
}

.svc-name {
  font-size: 16px;
  font-weight: 700;
  color: var(--text-primary);
  margin: 0;
}

.svc-desc {
  font-size: 13px;
  color: var(--text-secondary);
  margin: 0;
  line-height: 1.5;
}

.svc-custom {
  font-size: 13px;
  color: var(--text-muted);
  font-style: italic;
  border-left: 2px solid var(--accent);
  padding-left: 10px;
  margin-top: 4px;
}

.svc-footer {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-top: auto;
  padding-top: 12px;
  border-top: 1px solid var(--border);
}

.svc-details {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.svc-meta {
  font-size: 12px;
  color: var(--text-muted);
}

.svc-price {
  font-size: 20px;
  font-weight: 700;
  color: var(--text-primary);
}

.svc-cta {
  font-size: 12px;
  font-weight: 600;
  color: var(--accent);
  display: flex;
  align-items: center;
  margin-top: 8px;
  opacity: 0;
  transition: opacity 0.2s;
}

.service-card:hover .svc-cta {
  opacity: 1;
}

/* ── Responsive ── */
@media (max-width: 576px) {
  .hero-content {
    flex-direction: column;
    text-align: center;
    padding: 24px;
  }
  .provider-meta {
    justify-content: center;
  }
}
</style>