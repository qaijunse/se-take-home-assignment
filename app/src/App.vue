<template>
  <div id="app" class="p-2">
    <button class="btn btn-success mx-2" @click="addOrder(false)">
      Add Order
    </button>
    <button class="btn btn-success mx-2" @click="addOrder(true)">
      Add VIP Order
    </button>
    <button class="btn btn-success mx-2" @click="addBot(1)">+ Bot</button>
    <button class="btn btn-success mx-2" @click="addBot(2)">+ Bot (v2)</button>
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
          {{ orderLabel(order) }} ({{ order.countdown }})
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
        version: {
          countdown: {
            1: 10,
            2: 5,
          },
        }

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

  methods: {
    orderLabel(order) {
      return `${order.id} ${order.vip ? "(VIP)" : ""}`;
    },

    addOrder(vip = false) {
      this.orders.push({
        id: this.orderId++,
        status: this.status.PENDING,
        vip: vip,
        countdown: 0,
        timer: null,
      });

      if (vip) {
        this.orders = this.orders.sort((order1, order2) => {
          if (order1.vip && order2.vip) {
            return order1.id - order2.id;
          } else {
            return  order2.vip - order1.vip;
          }
        });
      }

      this.processOrders();
    },

    addBot(version) {
      if (this.bots.length >= this.config.bot.threshold) {
        return;
      }

      this.bots.push({
        order_id: null,
        timer: null,
        version: version,
      });

      //sort bot by version
      this.bots = this.bots.sort((bot1, bot2) => {
        return bot2.version - bot1.version;
      });

      this.processOrders();

    },

    removeBot() {
      var index = this.bots.length - 1;
      var bot = this.bots[index];
      if (bot.order_id) {
        var order = this.orders.find((order) => order.id == bot.order_id);
        order.status = this.status.PENDING;
        clearInterval(order.timer);
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

        order.countdown = this.config.bot.version.countdown[bot.version];
        order.timer = setInterval(() => {
          order.countdown--;
          if(order.countdown <= 0) {
            clearInterval(order.timer);
            order.status = this.status.COMPLETED;
            bot.order_id = null;
            bot.timer = null;

            this.processOrders();
          }
        }, 1000);
      }
    },

    processOrders() {
      this.bots.forEach((bot, index) => {
        this.processOrder(bot, index);
      });
    }
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
