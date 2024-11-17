<template>
  <div
    class="max-w-2xl mx-auto lg:max-w-2xl h-full flex flex-col lg:flex-row items-center gap-6 p-4"
  >
    <!-- Main Image -->
    <div class="w-full lg:w-1/2 h-[300px] md:h-[400px] lg:h-[500px] flex-shrink-0">
      <img
        class="h-full w-full rounded-lg object-cover"
        :src="product.product_img"
        alt="Product Image"
      />
    </div>

    <!-- Product Details -->
    <div class="w-full lg:w-1/2 text-left mt-4 lg:mt-0">
      <h2 class="text-xl md:text-2xl font-semibold mb-2">{{ product.name || 'Product Name' }}</h2>
      <p class="text-gray-700 mb-4 text-sm md:text-base">
        {{ product.description || 'This is a brief description of the product.' }}
      </p>

      <!-- Quantity Selector -->
      <div class="flex items-center gap-4 mb-4">
        <button
          @click="decreaseQuantity"
          class="px-2 py-1 text-white bg-gray-600 rounded-lg hover:bg-gray-700"
        >
          -
        </button>
        <span class="text-lg">{{ quantity }}</span>
        <button
          @click="increaseQuantity"
          class="px-2 py-1 text-white bg-gray-600 rounded-lg hover:bg-gray-700"
        >
          +
        </button>
      </div>

      <!-- Price -->
      <p class="text-lg md:text-xl font-semibold text-blue-600 mb-4">
        Price: ${{ product.price * quantity || 0 }}
      </p>

      <!-- Place Order Button -->
      <button
        @click="placeOrder"
        class="px-6 py-2 text-white bg-blue-700 rounded-lg hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
      >
        Place Order
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import coffeeService from '@/service/coffe.service';
import orderService from '@/service/order.service'; // Import the order service
import { useRouter } from 'vue-router';
// Props
const props = defineProps({
  id: {
    type: Number,
    required: true
  }
});

// Product and quantity states
const product = ref({});
const quantity = ref(1);
const router = useRouter();
// Fetch product data by ID
const fetchProductById = async () => {
  try {
    const response = await coffeeService.getCoffeeById(props.id);
    product.value = response.product;
  } catch (error) {
    console.error('Failed to fetch product data:', error);
  }
};

// Watch for changes in the ID prop
watch(() => props.id, fetchProductById);

// Fetch product data on component mount
onMounted(fetchProductById);

// Quantity control methods
const increaseQuantity = () => {
  quantity.value++;
};

const decreaseQuantity = () => {
  if (quantity.value > 1) quantity.value--;
};

const customer_id = localStorage.getItem('customer_id');
// Method to place an order
const placeOrder = async () => {
  const orderData = {
    customer_id: customer_id, // Replace with the actual customer ID if dynamic
    payment_method: 'Cash',
    product_id: props.id,
    quantity: quantity.value,
    price: product.value.price,
    total_price: product.value.price * quantity.value,
    status: false
  };

  try {
    const response = await orderService.createOrder(orderData);
    router.push('/Order');
  } catch (error) {
    console.error('Failed to place order:', error);
  }
};
</script>

<style>
/* Optional styling */
</style>
