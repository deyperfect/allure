<template>
    <div class="container my-5" style="max-width: 500px">
        <h1 class="text-center">Add New Product</h1>
        <form v-on:submit.prevent="handleSubmit">
            <div class="mb-3">
                <label for="productNameInput" class="form-label">Product Name</label>
                <input type="text" class="form-control" id="productNameInput" v-model="name" />
            </div>
            <div class="mb-3">
                <label for="productDescription" class="form-label">Description</label>
                <textarea
                    class="form-control"
                    id="productDescription"
                    v-model="description"
                    rows="5"
                ></textarea>
            </div>
            <div class="mb-3">
                <label for="productCategory" class="form-label">Category</label>
                <select
                    class="form-select"
                    id="productCategory"
                    v-model="category"
                >
                    <option disabled value="">Select category</option>
                    <option>In-Person</option>
                    <option>Virtual</option>
                    <option>Activity</option>
                    <option>Meal</option>
                    <option>Other</option>
                </select>
            </div>
            <button type="submit" class="btn btn-primary" v-if="isEnabled">Create</button>
            <button type="submit" class="btn btn-danger" disabled v-else>Create</button>
        </form>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import api from "../api";

const route = useRoute();
const router = useRouter();

const editMode = computed(() => !!route.params.id);

const form = ref({
  name: "",
  description: "",
  price: null,
  category: ""
});

const loading = ref(false);

const fetchProduct = async () => {
  try {
    const { data } = await api.get(`/products/${route.params.id}`);
    form.value = {
      name: data.name,
      description: data.description,
      price: data.price,
      category: data.category
    };
  } catch (err) {
    console.error("❌ Product not found:", err);
    router.push("/products/all");
  }
};

const submitProduct = async () => {
  try {
    loading.value = true;

    if (editMode.value) {
      await api.patch(`/products/${route.params.id}/update`, form.value);
    } else {
      await api.post("/products", form.value);
    }

    router.push("/products/all");

  } catch (err) {
    console.error("❌ Submit error:", err);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  if (editMode.value) {
    fetchProduct();
  }
});
</script>

