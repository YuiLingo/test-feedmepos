<template>
    <div class="row justify-content-center">
        <div class="col-2 text-center">
            <div class="btn-group-vertical d-grid gap-2 mx-2" role="group" aria-label="Vertical radio toggle button group">
                <button type="button" class="btn btn-primary" @click='addOrder("N")'>New Normal Order</button>
                <button type="button" class="btn btn-secondary" @click='addOrder("V")'>New VIP Order</button>
            </div>
        </div>
        <div class="col bg-light mx-1">
            <h4>PENDING</h4>
            <ol class="list-group list-group-numbered">
                <li v-for="order in orders" class="list-group-item d-flex justify-content-between align-items-start">
                    <div class="ms-2 me-auto">
                        <div class="fw-bold">{{ order.id }}</div>
                        {{ order.status }}
                    </div>
                </li>
            </ol>
        </div>
        <div class="col bg-light mx-1">
            <h4>COMPLETE</h4>
            <ol class="list-group list-group-numbered my-1">
                <li v-for="completedOrder in completedOrders"
                    class="list-group-item d-flex justify-content-between align-items-start bg-success bg-opacity-10">
                    <div class="ms-2 me-auto">
                        <div class="fw-bold">{{ completedOrder.id }}</div>
                        {{ completedOrder.status }}
                    </div>
                </li>
            </ol>
        </div>
    </div>
    <div class="row justify-content-center mt-5">
        <div class="col-2 text-center">
            <div class="btn-group-vertical d-grid gap-2 mx-2" role="group" aria-label="Vertical radio toggle button group">
                <button type="button" class="btn btn-primary" v-on:click='addBot()'>+ Bot</button>
                <button type="button" class="btn btn-primary" v-on:click='addBot("V")'>+ VIP Bot</button>
                <button type="button" class="btn btn-secondary" v-on:click='destroyBot()'>- Bot</button>
            </div>
        </div>
        <div class="col bg-light mx-1">
            <h4>Bots</h4>
            <ol class="list-group list-group-numbered my-1">
                <li v-for="bot in bots" class="list-group-item d-flex justify-content-between align-items-start">
                    <div class="ms-2 me-auto">
                        <div class="fw-bold">{{ bot.id }}</div>
                        {{ bot.orderId ? 'processing ' + bot.orderId : 'idle' }}
                    </div>
                    <span v-if="bot.countDown" class="badge bg-primary rounded-pill">{{ bot.countDown }}</span>
                </li>
            </ol>
        </div>
    </div>
</template>
<script>

export default {
    data() {
        return {
            bots: [], // {id, orderId, timer}
            orders: [], // {id, type, status}
            completedOrders: [], // {id, type, status}
        }
    },
    async created() {
        await this.checkBotAndAssignOrder();
    },
    methods: {
        addOrder(type) {
            let order = {
                id: type + Date.now(),
                type: type,
                status: 'pending',
            };

            if (type == 'V') {
                let lastIndex = -1;
                for (let i = 0; i < this.orders.length; i++) {
                    if (this.orders[i].status == 'pending') {
                        if (this.orders[i].type == type) {
                            lastIndex = i;
                        }
                    } else {
                        lastIndex = i;
                    }
                }
                this.orders.splice(lastIndex + 1, 0, order);
                this.checkBotAndAssignOrder();
                return;
            }

            this.orders.push(order);
            this.checkBotAndAssignOrder();
            return;
        },
        completeOrder(id) {
            let index = this.findOrderById(id);
            if (index !== -1) {
                this.completedOrders.push({
                    id: id,
                    type: this.orders[index].type,
                });
            }
            this.orders = this.orders.filter(order => order.id != id);
            return;
        },
        updateOrderStatusById(id, status) {
            let index = this.findOrderById(id);
            if (index !== -1) {
                this.orders[index].status = status;
            }
        },
        findOrderById(id) {
            return this.orders.findIndex(order => order.id == id);
        },
        addBot(type = 'N') {
            this.bots.push({
                id: Date.now() + '_' + type,
                orderId: 0,
                timer: null,
                countDown: 0,
                countDowmTimer: null,
                type: type,
            });
            this.checkBotAndAssignOrder();
        },
        destroyBot() {
            if (this.bots.length > 0 && this.bots[this.bots.length - 1].orderId) {
                clearTimeout(this.bots[this.bots.length - 1].timer);
                clearTimeout(this.bots[this.bots.length - 1].countDowmTimer);
                this.updateOrderStatusById(this.bots[this.bots.length - 1].orderId, 'pending')
            }
            this.bots.pop();
        },
        resetBot(id) {
            let index = this.findBotById(id);
            if (index !== -1) {
                clearTimeout(this.bots[index].timer);
                this.completeOrder(this.bots[index].orderId);
                this.bots[index].orderId = 0;
                this.bots[index].timer = null;
                this.bots[index].countDown = 0;
                this.bots[index].countDowmTimer = null;
            }
            this.checkBotAndAssignOrder();
            return;
        },
        findBotById(id) {
            return this.bots.findIndex(bot => bot.id == id);
        },
        async checkBotAndAssignOrder() {

            let pendingOrder = this.orders.find(order => order.status == 'pending');
            if (pendingOrder) {
                let pendingBot = this.bots.sort((botA, botB) => { return botA.type == 'V'? -1: 1;}).find(bot => bot.orderId == 0);

                if(pendingBot) {

                    let currentInstance = this;
                    currentInstance.updateOrderStatusById(pendingOrder.id, 'processing');
                    pendingBot.orderId = pendingOrder.id;
                    pendingBot.countDown = pendingBot.type == 'V' ? 5: 10;
                    currentInstance.countdown(pendingBot);
                }
            }

        },
        countdown(bot) {
            console.log(bot);
            let currentInstance = this;
            if (bot.countDown > 0) {
                bot.countDowmTimer = setTimeout(() => {
                    currentInstance.countdown(bot);
                }, 1000);
                bot.countDown--
            } else {
                currentInstance.resetBot(bot.id);
            }
        }
    }
}
</script>