<template>
  <div class="order-item">
    <div class="order-item__details">
      <p><strong>Заказ #</strong>{{ order.id }}</p>
      <p><strong>Объект:</strong> {{ order?.bid.objectName }}</p>
      <p><strong>Дата начала:</strong> {{ order.startDate }}</p>
      <p><strong>Дата конца:</strong> {{ order.endDate }}</p>
      <p><strong>Статус:</strong> {{ order.workStatus }}</p>
      <p><strong>Откликнувшиеся бригады:</strong></p>
      <ul>
        <li v-for="brigade in order.brigadeOrders" :key="brigade.id">
          {{ brigade.brigade.name }}
        </li>
      </ul>

      <p><strong>Привязанные материалы:</strong></p>
      <ul v-if="attachedMaterials.length">
        <li v-for="material in attachedMaterials" :key="material.id">
          {{ material.name }} ({{ material.quantity }} {{ material.measurementUnit }})
        </li>
      </ul>
      <p v-else>Нет привязанных материалов.</p>
    </div>

    <div class="order-item__actions" v-if="order.workStatus !== 'Готово'">
      <button
          class="action-button"
          :disabled="isOrderModalOpen"
          @click="openOrderModal(order)"
      >
        Изменить
      </button>
      <button @click="$router.push({ name: 'MaterialForOrder', params: { orderId: order.id } })">
        Материалы к заказу
      </button>
    </div>

    <edit-order-status
        :show="isOrderModalOpen"
        :order="selectedOrder"
        :bids="availableBids"
        @save="handleOrderSave"
        @close="closeOrderModal"
    />

  </div>
</template>



<script>
import axios from "axios";
import EditOrderStatus from "@/components/EditOrderStatus.vue";

export default {
  components: {EditOrderStatus},
  props: {
    order: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      availableBids: [],
      selectedOrder: null,
      isOrderModalOpen: false,
      attachedMaterials: [], // Список привязанных материалов
    };
  },
  mounted() {
    this.fetchAttachedMaterials(); // Загрузка привязанных материалов при инициализации компонента
  },
  methods: {
    closeOrderModal() {
      this.isOrderModalOpen = false;
    },
    handleOrderSave(updatedOrder) {
      console.log("Обновленный заказ:", updatedOrder);
      const payload = updatedOrder.workStatus; // Передаем только статус как строку

      axios
          .patch(`https://localhost:7265/Orders/${this.selectedOrder.id}/status`, payload, {
            headers: {
              "Content-Type": "application/json", // Указываем формат данных
            },
          })
          .then((response) => {
            console.log("Изменения заказа сохранены:", response.data);
            this.closeOrderModal();
          })
          .catch((error) => {
            console.error("Ошибка при сохранении заказа:", error);
          });
    },
    openOrderModal(order) {
      if (!order) {
        console.error("Заказ не найден");
        return;
      }
      this.selectedOrder = { ...order }; // Создаем копию заказа
      this.isOrderModalOpen = true; // Открываем модальное окно
    },
    async fetchAttachedMaterials() {
      try {
        const response = await axios.get(`https://localhost:7265/Orders/${this.order.id}/Materials`);
        this.attachedMaterials = response.data;
      } catch (error) {
        console.error("Ошибка загрузки привязанных материалов:", error.message);
      }
    },
  },
};
</script>






<style scoped>
.status-готово {
  color: green;
  font-weight: bold;
}
.status-выполняется {
  color: orange;
}
.status-начато {
  color: blue;
}

/* Основной стиль компонента */
.order-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
}

.order-item__details {
  flex-grow: 1;
}

.order-item__actions {
  display: flex;
  gap: 10px;
}

.action-button {
  padding: 5px 10px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 3px;
  cursor: pointer;
}

.action-button:hover {
  background-color: #0056b3;
}

.action-button.cancel {
  background-color: #dc3545;
}

.action-button.cancel:hover {
  background-color: #a71d2a;
}

/* Модальное окно */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 300px;
  text-align: center;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  animation: fadeIn 0.3s ease-out;
}

/* Анимация появления модального окна */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-actions {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}
</style>
