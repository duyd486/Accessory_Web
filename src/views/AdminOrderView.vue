<template>
    <body class="bg-light">
        <div class="container-fluid">
            <div class="row">
                <AdminSidebarComponent />

                <div class="col-9 col-md-10 p-4">
                    <AdminHeaderComponent />
                    <hr />

                    <div class="d-flex justify-content-between align-items-center mb-2">
                        <h3 class="fw-bold">Quản lý Đơn hàng</h3>

                        <div class="d-flex flex-wrap gap-2 justify-content-end">
                            <button
                                type="button"
                                class="btn btn-primary btn-sm rounded-pill px-3 d-inline-flex align-items-center gap-2 shadow-sm"
                                title="Đồng bộ đơn hàng từ Lazada"
                            >
                                <Icon icon="simple-icons:lazada" width="16" height="16" />
                                <span class="fw-semibold">Đồng bộ đơn hàng Lazada</span>
                            </button>
                            <button
                                type="button"
                                class="btn btn-warning btn-sm rounded-pill px-3 d-inline-flex align-items-center gap-2 text-white shadow-sm"
                                title="Đồng bộ đơn hàng từ Shopee"
                            >
                                <Icon icon="simple-icons:shopee" width="16" height="16" />
                                <span class="fw-semibold">Đồng bộ đơn hàng Shopee</span>
                            </button>
                        </div>
                    </div>
                    <p class="text-muted mb-4">Kiểm duyệt cập nhật đơn hàng</p>

                    <div class="row g-3 mb-4 border rounded p-2">
                        <div class="col-md-4">
                            <p class="fw-semibold mb-1">Tìm kiếm</p>
                            <input
                                type="text"
                                class="form-control"
                                v-model="search"
                                placeholder="🔍 Mã đơn hàng"
                            />
                        </div>
                        <div class="col-md-4">
                            <p class="fw-semibold mb-1">Trạng thái</p>
                            <select class="form-select" v-model="filterStatus">
                                <option value="" selected>Tất cả trạng thái</option>
                                <option value="3">Chuẩn bị hàng</option>
                                <option value="4">Chuẩn bị hàng</option>
                                <option value="5">Đang giao</option>
                                <option value="6">Đã hoàn thành</option>
                            </select>
                        </div>
                    </div>

                    <div class="table-responsive row g-3 mb-4 border rounded p-3">
                        <table
                            class="table table-bordered table-hover border align-middle bg-white rounded shadow-sm"
                        >
                            <thead class="text-center">
                                <tr>
                                    <th>Mã đơn</th>
                                    <th>Ngày đặt</th>
                                    <th>Tên sản phẩm</th>
                                    <th>Người đặt</th>
                                    <th>Tổng tiền</th>
                                    <th>Trạng thái</th>
                                    <th>Hành động</th>
                                </tr>
                            </thead>
                            <tbody class="text-center">
                                <tr v-for="order in filteredOrders" :key="order.id">
                                    <td class="fw-bold">#{{ order.order_code }}</td>

                                    <td>{{ formatDate(order.created_at) }}</td>

                                    <td class="text-start">
                                        <div
                                            v-for="product in order.products"
                                            :key="product.product_id"
                                        >
                                            - {{ product.product_name }}
                                            <span class="text-muted"
                                                >(SL: {{ product.quantity }})</span
                                            >
                                        </div>
                                    </td>

                                    <td>{{ order.user_name }}</td>

                                    <td class="fw-bold text-danger">
                                        {{ formatCurrency(order.total_price) }}
                                    </td>

                                    <td>
                                        <select
                                            v-model="order.status"
                                            class="form-select text-center fw-semibold"
                                            :class="statusClass(order.status)"
                                        >
                                            <option :value="3">Chuẩn bị hàng</option>
                                            <option :value="4">Chuẩn bị hàng</option>
                                            <option :value="5">Giao hàng</option>
                                            <option :value="6">Đã hoàn thành</option>
                                        </select>
                                    </td>
                                    <td>
                                        <button
                                            class="btn btn-success btn-sm"
                                            @click="updateOrderStatus(order)"
                                        >
                                            <i class="bi bi-save"></i> Lưu
                                        </button>
                                    </td>
                                </tr>

                                <tr v-if="filteredOrders.length === 0">
                                    <td colspan="7" class="text-muted py-4">
                                        Không tìm thấy đơn hàng nào phù hợp.
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </body>
</template>

<script setup>
import axios from 'axios'
import { Icon } from '@iconify/vue'
import AdminSidebarComponent from '@/components/AdminSidebarComponent.vue'
import AdminHeaderComponent from '@/components/AdminHeaderComponent.vue'
import { apiHelper } from '@/helpers/axios'
</script>

<script>
export default {
    components: {
        AdminSidebarComponent,
        AdminHeaderComponent,
    },

    data() {
        return {
            orders: [],
            search: '',
            filterStatus: '', // Biến dùng để lọc
            token: sessionStorage.getItem('token'),
        }
    },

    created() {},

    mounted() {
        this.listOrders()
    },

    computed: {
        // Hàm lọc tự động chạy khi search hoặc filterStatus thay đổi
        filteredOrders() {
            return this.orders.filter((order) => {
                // Lọc theo mã đơn hàng
                const matchCode =
                    this.search === '' ||
                    order.order_code.toString().toLowerCase().includes(this.search.toLowerCase())

                // Lọc theo trạng thái (so sánh lỏng == để khớp string '4' với số 4)
                const matchStatus = this.filterStatus === '' || order.status == this.filterStatus

                return matchCode && matchStatus
            })
        },
    },

    methods: {
        listOrders() {
            try {
                apiHelper
                    .get('/list-orders', {
                        headers: {
                            Authorization: `Bearer ${this.token}`,
                        },
                    })
                    .then((res) => {
                        if (res.status === 200) {
                            this.orders = res.data.data
                        }
                    })
                    .catch((error) => {
                        console.log(error)
                    })
            } catch (error) {
                console.log(error)
            }
        },
        updateOrderStatus(order) {
            try {
                apiHelper
                    .post(
                        '/update-order-status',
                        {
                            order_id: order.id,
                            status: order.status,
                        },
                        {
                            headers: {
                                Authorization: `Bearer ${this.token}`,
                            },
                        },
                    )
                    .then((res) => {
                        if (res.status === 200) {
                            alert('Cập nhật trạng thái đơn hàng thành công')
                        }
                    })
                    .catch((error) => {
                        console.log(error)
                        alert('Cập nhật thất bại')
                    })
            } catch (error) {
                console.log(error)
            }
        },
        formatDate(date) {
            if (!date) return ''
            return new Date(date).toLocaleDateString('vi-VN', {
                day: '2-digit',
                month: '2-digit',
                year: 'numeric',
                hour: '2-digit',
                minute: '2-digit',
            })
        },
        statusClass(status) {
            switch (Number(status)) {
                case 4:
                    return 'status-pending'
                case 5:
                    return 'status-shipping'
                case 6:
                    return 'status-done'
                default:
                    return ''
            }
        },
        formatCurrency(amount) {
            if (!amount) return '0 ₫'
            return new Intl.NumberFormat('vi-VN', {
                style: 'currency',
                currency: 'VND',
            }).format(amount)
        },
    },
}
</script>

<style scoped>
.status-pending {
    background-color: #fff3cd !important;
    color: #856404 !important;
    border: 1px solid #ffeeba !important;
}

.status-shipping {
    background-color: #cfe2ff !important;
    color: #084298 !important;
    border: 1px solid #b6d4fe !important;
}

.status-done {
    background-color: #d1e7dd !important;
    color: #0f5132 !important;
    border: 1px solid #badbcc !important;
}
</style>
