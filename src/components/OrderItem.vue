<template>
  <div class="order-item">
    <div class="order-item__details">
      <p><strong>Заказ #</strong> {{ order.id }}</p>
      <p><strong>Дата начала:</strong> {{ order.startDate }}</p>
      <p><strong>Дата конца:</strong> {{ order.endDate }}</p>
      <p><strong>Статус:</strong> {{ order.workStatus }}</p>
      <p><strong>Откликнувшиеся бригады:</strong></p>
      <ul>
        <li v-for="brigade in order.brigadeOrders" :key="brigade.id">
          {{ brigade.brigade.name }}
        </li>
      </ul>
    </div>


    <div class="order-item__actions">
      <button class="action-button" @click="openModal">Откликнуться</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  props: {
    order: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      brigades: [], // Список бригад
    };
  },
  methods: {
    async openModal() {
      try {
        const response = await axios.get("https://localhost:7265/Brigades");
        this.brigades = response.data;

        // Передаем данные через query
        this.$router.push({
          name: "brigade-selection",
          query: { brigades: JSON.stringify(this.brigades), orderId: this.order.id },
        });


      } catch (error) {
        console.error("Ошибка при загрузке бригад:", error);
        alert("Не удалось загрузить список бригад.");
      }
    },

  },
};
</script>




<style scoped>
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
