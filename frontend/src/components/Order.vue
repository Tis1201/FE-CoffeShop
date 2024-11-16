<template>
  <div class="bg-white">
    <div class="mb-10">
      <section id="page-header">
        <h2>#stayhome</h2>
        <p>Save more with coupons & up to 70% off!</p>
      </section>
    </div>
    <div class="relative overflow-x-auto shadow-md sm:rounded-lg mb-10">

      <table class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400">
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th
              scope="col"
              class="p-4"
            >
              <div class="flex items-center">
                <!-- Checkbox chọn tất cả -->
                <input
                  id="checkbox-all-search"
                  type="checkbox"
                  v-model="selectAll"
                  @change="toggleAll"
                  class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 dark:focus:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600"
                />
                <label
                  for="checkbox-all-search"
                  class="sr-only"
                  >checkbox</label
                >
              </div>
            </th>
            <th
              scope="col"
              class="px-6 py-3"
            >
              Product name
            </th>
            <th
              scope="col"
              class="px-6 py-3"
            >
              Qty
            </th>
            <th
              scope="col"
              class="px-6 py-3"
            >
              Payment method
            </th>
            <th
              scope="col"
              class="px-6 py-3"
            >
              Price
            </th>
            <th
              scope="col"
              class="px-6 py-3"
            >
              Total price
            </th>
            <th
              scope="col"
              class="px-6 py-3"
            >
              Action
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in order"
            :key="item.order_item_id"
            class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-600"
          >
            <td class="w-4 p-4">
              <div class="flex items-center">
                <input
                  :id="'checkbox-table-' + item.order_item_id"
                  type="checkbox"
                  v-model="selectedOrders"
                  :value="item"
                  class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 dark:focus:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600"
                />
                <label
                  :for="'checkbox-table-' + item.order_item_id"
                  class="sr-only"
                  >checkbox</label
                >
              </div>
            </td>
            <th
              scope="row"
              class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white"
            >
              {{ item.product_name }}
            </th>
            <td class="px-6 py-4">{{ item.quantity }}</td>
            <td class="px-6 py-4">{{ item.payment_method }}</td>
            <td class="px-6 py-4">{{ item.price }}</td>
            <td class="px-6 py-4">{{ item.total_price }}</td>
            <td class="px-6 py-4">
              <a
                href="#"
                @click.prevent="handleDelete(item.order_item_id)"
                class="font-medium text-blue-600 dark:text-blue-500 hover:underline"
                >Delete</a
              >
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div
      v-if="selectedOrders.length > 0"
      class="relative overflow-x-auto max-w-md mx-auto shadow-lg rounded-lg border border-gray-200 dark:border-gray-700 mb-20"
    >
      <table class="w-full text-sm text-left rtl:text-right text-gray-700 dark:text-gray-300">
        <thead class="text-xs font-semibold text-white uppercase bg-coffee dark:bg-blue-800">
          <tr>
            <th
              scope="col"
              class="px-4 py-3 rounded-tl-lg"
            >
              Product name
            </th>
            <th
              scope="col"
              class="px-4 py-3"
            >
              Qty
            </th>
            <th
              scope="col"
              class="px-4 py-3 rounded-tr-lg"
            >
              Price
            </th>
          </tr>
        </thead>
        <tbody class="bg-gray-50 dark:bg-gray-800">
          <tr
            v-for="item in selectedOrders"
            :key="item.id"
            class="border-b border-gray-200 dark:border-gray-700"
          >
            <th
              scope="row"
              class="px-4 py-3 font-medium text-gray-900 dark:text-white"
            >
              {{ item.product_name }}
            </th>
            <td class="px-4 py-3">{{ item.quantity }}</td>
            <td class="px-4 py-3">{{ item.price }}</td>
          </tr>
        </tbody>
        <tfoot>
          <tr class="font-semibold text-gray-900 dark:text-white bg-gray-100 dark:bg-gray-700">
            <th
              scope="row"
              class="px-4 py-3 text-base"
            >
              Total
            </th>
            <td class="px-4 py-3">{{ totalQuantity }}</td>
            <td class="px-4 py-3">{{ totalPrice }}</td>
          </tr>
        </tfoot>
      </table>
      <!-- Nút thanh toán -->
      <div class="flex justify-center my-4">
        <button
          @click="processPayment"
          class="px-6 py-2 text-white bg-blue-600 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          Thanh toán
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import orderService from '@/service/order.service';

// Props
const props = defineProps({
  order: {
    type: Array,
    required: true
  }
});

// Emit events for notifications
const notify = defineEmits(['paymentSuccess', 'deleteSuccess']); // Renamed from `emit` to `notify`

// State for selecting items
const selectAll = ref(false);
const selectedOrders = ref([]);

// Toggle all selections
const toggleAll = () => {
  if (selectAll.value) {
    selectedOrders.value = [...props.order];
  } else {
    selectedOrders.value = [];
  }
};

// Computed totals
const totalQuantity = computed(() =>
  selectedOrders.value.reduce((acc, item) => acc + item.quantity, 0)
);
const totalPrice = computed(() =>
  selectedOrders.value.reduce((acc, item) => acc + item.total_price, 0)
);

// Handle delete by order_item_id
const handleDelete = async (order_item_id) => {
  try {
    await orderService.deleteOrder(order_item_id);
    notify('deleteSuccess'); // Emit delete success event
    // Remove the item from the main order list and selectedOrders after deletion
    const index = props.order.findIndex((item) => item.order_item_id === order_item_id);
    if (index !== -1) {
      props.order.splice(index, 1);
    }
    // Remove from selectedOrders if it exists there
    const selectedIndex = selectedOrders.value.findIndex(
      (item) => item.order_item_id === order_item_id
    );
    if (selectedIndex !== -1) {
      selectedOrders.value.splice(selectedIndex, 1);
    }
  } catch (error) {
    console.error('Failed to delete order:', error);
    alert('Failed to delete order. Please try again.');
  }
};

const processPayment = async () => {
  try {
    for (const item of selectedOrders.value) {
      // Update each selected order
      await orderService.updateOrder(item.order_item_id, {
        status: true // Example status to mark as paid
      });
    }
    selectedOrders.value = []; 
    notify('paymentSuccess'); 
    setTimeout(() => {
      window.location.reload();
    }, 3000); 
  } catch (error) {
    console.error('Failed to process payment:', error);
    alert('Failed to update orders. Please try again.');
  }
};
</script>
