<template>
  <div class="container">
    <VueLoading :active="isLoading">
      <img src="@/assets/images/loading-spinner.gif" alt="VueLoadingSpinner" />
    </VueLoading>
    <table class="table mt-4">
      <thead>
        <tr>
          <th>購買時間</th>
          <th>Email</th>
          <th>購買款項</th>
          <th>應付金額</th>
          <th>是否付款</th>
          <th>編輯</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="item in orders" :key="item.id">
          <!-- {{order}} -->
          <tr v-if="orders.length">
            <td>{{ $filters.date(item.create_at) }}</td>
            <td>
              <span> {{ item.user.email }} </span>
            </td>
            <td>
              <ul class="px-0 list-unstyled">
                <li v-for="(singleItem, index) in item.products" :key="index">
                  {{ singleItem.product.title }} / 數量：{{ singleItem.qty }}
                  {{ singleItem.product.unit }}
                </li>
              </ul>
            </td>
            <td class="text-right">{{ item.total }}</td>
            <!-- switch toggle -->
            <td>
              <div class="form-check form-switch">
                <input
                  class="form-check-input"
                  type="checkbox"
                  v-model="item.is_paid"
                  @change="switchPaidStatus(item)"
                />
                <label class="form-check-label">
                  <span class="text-success" v-if="item.is_paid === true"
                    >已付款</span
                  >
                  <span v-else>未付款</span>
                </label>
              </div>
            </td>
            <!-- switch toggle -->
            <td>
              <div class="btn-group">
                <button
                  class="btn btn-outline-secondary btn-sm"
                  type="button"
                  @click="openOrderModal('checkOrder', item)"
                >
                  檢視
                </button>
                <button
                  class="btn btn-outline-danger btn-sm"
                  type="button"
                  @click="openOrderModal('deleteOrder', item)"
                >
                  刪除
                </button>
              </div>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
    <!-- 檢視訂單 order modal -->
    <OrderModal
      ref="orderModal"
      :order="tempOrderData"
      :orderModal="OrderModal"
      @get-order-data="getOrderData"
    ></OrderModal>
    <!-- 刪除訂單 delete modal  -->
    <DeleteOrderModal
      :itemData="tempOrderData"
      :deleteOrderModal="DeleteOrderModal"
      @get-order-data="getOrderData"
    ></DeleteOrderModal>
    <!-- 產品分頁元件 -->
    <DashboardPagination
      :pages="pagination"
      @get-product="getOrderData"
    ></DashboardPagination>
  </div>
</template>

<script>
// 引入 modal 元件
import BsProductModal from 'bootstrap/js/dist/modal'
// 引入自己的元件
import DashboardPagination from '@/components/DashboardPagination.vue'
import OrderModal from '@/components/OrderModal.vue'
import DeleteOrderModal from '@/components/DeleteOrderModal.vue'
export default {
  data () {
    return {
      isLoading: false,
      // 原始的完整資料
      orders: {},
      // 複製給 modal 用的資料
      tempOrderData: {},
      pagination: {},
      // 引入自己寫的 BS 元件
      OrderModal: {},
      DeleteOrderModal: {},
      // toggle 狀態
      isPaid: true
    }
  },
  components: {
    DashboardPagination,
    OrderModal,
    DeleteOrderModal
  },
  methods: {
    switchPaidStatus () {
      this.$refs.orderModal.updatePaidData()
    },
    // 取得訂單資料
    getOrderData (currentPage = 1) {
      this.isLoading = true
      this.$http
        .get(
          `${process.env.VUE_APP_API}/v2/api/${process.env.VUE_APP_PATH}/admin/orders/?page=${currentPage}`
        )
        .then((res) => {
          this.isLoading = false
          this.orders = res.data.orders
          this.pagination = res.data.pagination
        })
        .catch((err) => {
          console.log(err.data)
        })
    },
    openOrderModal (status, tempOrderObj) {
      if (status === 'checkOrder') {
        this.tempOrderData = { ...tempOrderObj }
        this.OrderModal.show()
      } else if (status === 'deleteOrder') {
        this.tempOrderData = { ...tempOrderObj }
        this.DeleteOrderModal.show()
      }
    }
  },
  mounted () {
    this.getOrderData()
    this.OrderModal = new BsProductModal(document.getElementById('orderModal'))
    this.DeleteOrderModal = new BsProductModal(
      document.getElementById('deleteOrderModal')
    )
  }
}
</script>
