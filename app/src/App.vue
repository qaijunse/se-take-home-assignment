<template>
  <div id="app" class="p-2">
    <button class="btn btn-success mx-2" @click="addOrder(false)">
      Add Order
    </button>
    <button class="btn btn-success mx-2" @click="addOrder(true)">
      Add VIP Order
    </button>
    <button class="btn btn-success mx-2" @click="addBot()">+ Bot</button>
    <button class="btn btn-success mx-2" @click="removeBot()">- Bot</button>

    <h5 class="my-5">Total Bot: {{ bots.length }}</h5>
    <div class="row">
      <div class="col">
        Pending ({{ pendingOrder.length }})
        <p v-for="order in pendingOrder" :key="order.id">
          {{ orderLabel(order) }}
        </p>
      </div>
      <div class="col">
        Processing ({{ processingOrder.length }})
        <p v-for="order in processingOrder" :key="order.id">
          {{ orderLabel(order) }}
        </p>
      </div>
      <div class="col">
        Comepleted ({{ completedOrder.length }})
        <p v-for="order in completedOrder" :key="order.id">
          {{ orderLabel(order) }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "OrderView",
  components: {},

  data: () => ({
    config: {
      bot: {
        threshold: 20,
      },
    },
    orderId: 1,
    orders: [],
    bots: [],
    status: {
      PENDING: 0,
      PROCESSING: 1,
      COMPLETED: 2,
    },
  }),

  mounted() {
    setInterval(() => {
      this.bots.forEach((bot, index) => {
        this.processOrder(bot, index);
      });
    }, 500);
  },

  methods: {
    orderLabel(order) {
      return `${order.id} ${order.vip ? "(VIP)" : ""}`;
    },

    addOrder(vip = false) {
      this.orders.push({
        id: this.orderId++,
        status: this.status.PENDING,
        vip: vip,
      });

      if (vip) {
        this.orders = this.orders.sort((order1, order2) => {
          if (order1.vip && order2.vip) {
            return order1.id - order2.id;
          } else {
            if (order1.vip && order2.vip == false) {
              return -1;
            } else {
              return 1;
            }
          }
        });
      }
    },

    addBot() {
      if (this.bots.length >= this.config.bot.threshold) {
        return;
      }

      this.bots.push({
        order_id: null,
        timer: null,
      });
    },

    removeBot() {
      var index = this.bots.length - 1;
      var bot = this.bots[index];
      if (bot.order_id) {
        var order = this.orders.find((order) => order.id == bot.order_id);
        order.status = this.status.PENDING;
        clearTimeout(bot.timer);
      }

      this.bots.splice(index, 1);
    },

    processOrder(bot) {
      const order = this.pendingOrder[0];

      if (bot.order_id) {
        return;
      }

      if (order != null) {
        bot.order_id = order.id;
        order.status = this.status.PROCESSING;
        bot.timer = setTimeout(() => {
          order.status = this.status.COMPLETED;
          bot.order_id = null;
          bot.timer = null;
        }, 1000 * 10);
      }
    },
  },

  computed: {
    pendingOrder() {
      return this.orders.filter((order) => {
        return order.status === this.status.PENDING;
      });
    },
    processingOrder() {
      return this.orders.filter((order) => {
        return order.status === this.status.PROCESSING;
      });
    },
    completedOrder() {
      return this.orders.filter((order) => {
        return order.status === this.status.COMPLETED;
      });
    },
  },

  filters: {
    orderLabel(val) {
      return `${val.id} ${val.vip == true ? "(VIP)" : ""}`;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
