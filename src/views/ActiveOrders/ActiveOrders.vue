<template>
  <div
    :class="{ 'w-220': expandMode }"
    class="active-orders pt-3 lg:pt-0"
  >
    <LoadingContainer
      :loading="isFetching"
      class="w-full"
    >
      <ActiveOrdersTable
        v-if="filteredItems.length > 0"
        :table-headers="tableHeaders"
        :items="filteredItems"
        :expanded="expandMode"
        @remove-order="confirmRemove"
      />
      <div
        v-if="filteredItems.length === 0"
        class="noActiveOrders"
      >
        <p>
          No Active Orders
        </p>
      </div>
    </LoadingContainer>
    <ConfirmModal
      :show="showRemoveModal"
      :pending="orderRemoving"
      type="type"
      title="confirm order remove"
      @close="stopRemovingOrder"
      @confirm="removeOrder">
      <div class="color-text-primary">Are you sure you want to remove order?</div>
    </ConfirmModal>
  </div>
</template>
<script>
import Vue from 'vue'
import { mapGetters, mapActions } from 'vuex'
import LoadingContainer from '@/components/LoadingContainer'
import ActiveOrdersTable from './ActiveOrdersTable'
import ConfirmModal from '@/views/ConfirmModal/ConfirmModal.vue'

export default {
  name: 'ActiveOrders',
  components: {
    ActiveOrdersTable,
    LoadingContainer,
    ConfirmModal
  },
  props: {
    expandMode: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      showRemoveModal: false,
      orderForCancel: {},
      orderRemoving: false
    }
  },
  computed: {
    ...mapGetters({
      items: 'activeOrders/getActiveList',
      searchStr: 'activeOrders/getSearchStr',
      isFetching: 'activeOrders/isFetching'
    }),
    filteredItems() {
      if (!this.items) {
        return []
      }
      const search = this.searchStr.toLowerCase()
      return this.items.filter(item => {
        return item.payAssetSymbol && item.payAssetSymbol.toLowerCase().includes(search)
      })
    },
    tableHeaders() {
      if (this.expandMode) {
        return [
          { title: 'Pair', field: 'payAssetSymbol', align: 'left' },
          { title: 'Price', field: 'price', align: 'left' },
          { title: 'Get', field: 'get', align: 'left' },
          { title: 'Spend', field: 'spend', align: 'left' },
          { title: 'Filled', field: 'filled', align: 'right', expanded: true },
          { title: 'Expiring', field: 'dateExpiring', align: 'right' }
        ]
      }
      return [
        { title: 'Pair', field: 'payAssetSymbol', align: 'left' },
        { title: 'Price', field: 'price', align: 'left' },
        { title: 'Vol./Filled', field: 'filled', align: 'right' }
      ]
    }
  },
  methods: {
    ...mapActions({
      cancelOrder: 'transactions/cancelOrder',
      updateUser: 'acc/fetchCurrentUser'
    }),
    async confirmRemove(order) {
      const unlocked = await this.$unlock()
      if (unlocked) {
        this.orderForCancel = order
        this.showRemoveModal = true
      }
    },
    async removeOrder() {
      this.orderRemoving = true
      const { orderId } = this.orderForCancel
      const res = await this.cancelOrder({ orderId })
      this.orderRemoving = false
      if (res.success) {
        Vue.prototype.$toast.success('Order canceled')
        await this.updateUser()
      } else {
        Vue.prototype.$toast.error(res.error)
      }
      this.orderForCancel = {}
      this.showRemoveModal = false
    },
    stopRemovingOrder() {
      this.orderForCancel = {}
      this.showRemoveModal = false
    }
  }
}
</script>
<style lang="scss">
  .active-orders {
    position: relative;
    height: 100%;
    flex-grow: 1;
    display: flex;
  }

  .noActiveOrders {
    display: flex;
    height: 100%;
    align-items: center;
    p {
      color:config('colors.white');
      opacity: 0.5;
      text-align: center;
      width: 100%;
      font-size: 1.5rem;
    }
  }
</style>
