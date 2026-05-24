<template>
    <div class="content-box">
        <ul class="order-tabs d-flex mb-4">
            <li :class="{ active: orderTab === 'preparing' }" @click="orderTab = 'preparing'">
                Chuẩn bị hàng
            </li>

            <li :class="{ active: orderTab === 'shipping' }" @click="orderTab = 'shipping'">
                Đang giao hàng
            </li>

            <li :class="{ active: orderTab === 'completed' }" @click="orderTab = 'completed'">
                Đã hoàn thành
            </li>
        </ul>

        <div v-if="orderTab === 'preparing'">
            <div v-for="(item, index) in orderPreparing" :key="index" class="order-wrapper p-3 border rounded mb-3">
                <div class="d-flex justify-content-between align-items-center mb-3">
                    <div class="d-flex align-items-center">
                        <img :src="item.category_thumbnail" class="category-thumb me-2" />
                        <div class="me-3">
                            <h5 class="mb-0">{{ item.category_title }}</h5>
                        </div>
                        <button class="btn btn-outline-primary btn-sm">Xem danh mục</button>
                    </div>

                    <span class="order-status" :class="item.bill_status === 3 ? 'paid' : 'status-direct'">
                        {{ item.bill_status === 3 ? 'ĐÃ THANH TOÁN' : 'THANH TOÁN TRỰC TIẾP' }}
                    </span>
                </div>

                <div class="order-item d-flex p-3 border rounded">
                    <img :src="item.product_thumbnail" class="image-rounded me-3" />

                    <div class="flex-grow-1">
                        <h6>{{ item.product_name }}</h6>
                        <p class="text-muted">Danh mục: {{ item.category_title }}</p>
                        <p>x{{ item.detail_quantity }}</p>
                    </div>

                    <div class="text-end">
                        <p class="text-danger fw-bold">
                            {{ (item.product_price * item.detail_quantity).toLocaleString() }}đ
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="orderTab === 'shipping'">
            <div v-for="(item, index) in orderShipping" :key="index" class="order-wrapper p-3 border rounded mb-3">
                <div class="d-flex justify-content-between align-items-center mb-3">
                    <div class="d-flex align-items-center">
                        <img :src="item.category_thumbnail" class="category-thumb me-2" />
                        <div class="me-3">
                            <h5 class="mb-0">{{ item.category_title }}</h5>
                        </div>
                        <button class="btn btn-outline-primary btn-sm">Xem danh mục</button>
                    </div>

                    <span class="order-status status-shipping">ĐANG GIAO</span>
                </div>

                <div class="order-item d-flex p-3 border rounded">
                    <img :src="item.product_thumbnail" class="image-rounded me-3" />

                    <div class="flex-grow-1">
                        <h6>{{ item.product_name }}</h6>
                        <p class="text-muted">Danh mục: {{ item.category_title }}</p>
                        <p>x{{ item.detail_quantity }}</p>
                    </div>

                    <div class="text-end">
                        <p class="text-danger fw-bold">
                            {{ (item.product_price * item.detail_quantity).toLocaleString() }}đ
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="orderTab === 'completed'">
            <div v-for="(item, index) in orderCompleted" :key="index" class="order-wrapper p-3 border rounded mb-3">
                <div class="d-flex justify-content-between align-items-center mb-3">
                    <div class="d-flex align-items-center">
                        <img :src="item.category_thumbnail" class="category-thumb me-2" />
                        <div class="me-3">
                            <h5 class="mb-0">{{ item.category_title }}</h5>
                        </div>
                        <button class="btn btn-outline-primary btn-sm">Xem danh mục</button>
                    </div>

                    <span class="order-status paid">HOÀN THÀNH</span>
                </div>

                <div class="order-item d-flex p-3 border rounded">
                    <img :src="item.product_thumbnail" class="image-rounded me-3" />

                    <div class="flex-grow-1">
                        <h6>{{ item.product_name }}</h6>
                        <p class="text-muted">Danh mục: {{ item.category_title }}</p>
                        <p>x{{ item.detail_quantity }}</p>
                    </div>

                    <div class="text-end">
                        <p class="text-danger fw-bold">
                            {{ (item.product_price * item.detail_quantity).toLocaleString() }}đ
                        </p>
                        <div class="d-flex gap-2 justify-content-end">
                            <button class="btn btn-outline-secondary btn-sm" @click="openFeedback(item.bill_id)">
                                Đánh giá
                            </button>
                            <button class="btn btn-outline-primary btn-sm" @click="buyNow(item)">
                                Mua lại
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <BillReviewModal :isVisible="showFeedbackModal" :billId="selectedBillId" @close="showFeedbackModal = false" />
    </div>
</template>

<script>
import { apiHelper } from '@/helpers/axios'
// 1. IMPORT COMPONENT
import BillReviewModal from './BillReviewModal.vue'

export default {
    // 2. REGISTER COMPONENT
    components: {
        BillReviewModal
    },
    data() {
        return {
            orderTab: 'completed',
            orderPreparing: [],
            orderShipping: [],
            orderCompleted: [],

            // 3. KHAI BÁO BIẾN STATE
            showFeedbackModal: false,
            selectedBillId: null
        }
    },

    async created() {
        const params = new URLSearchParams(window.location.search)
        const orderCode = params.get('orderCode')
        const tab = params.get('tab')

        if (tab) this.orderTab = tab

        if (orderCode) {
            try {
                const token = sessionStorage.getItem('token')
                await apiHelper.get(`/check-payment-status?orderCode=${orderCode}`, {
                    headers: { Authorization: `Bearer ${token}` },
                })
                window.location.replace(window.location.pathname + '?tab=completed')
            } catch (e) {
                console.error(e)
            }
        }

        this.orderHistory()
    },

    methods: {
        async orderHistory() {
            try {
                const token = sessionStorage.getItem('token')
                if (!token) return

                const res = await apiHelper.get('/order-history', {
                    headers: { Authorization: `Bearer ${token}` },
                })

                if (res.status === 200 && res.data?.data) {
                    this.orderPreparing = res.data.data.orderPreparing || []
                    this.orderShipping = res.data.data.orderShipping || []
                    this.orderCompleted = res.data.data.orderCompleted || []
                }
            } catch (error) {
                console.error(error)
            }
        },

        buyNow(item) {
            const token = sessionStorage.getItem('token')

            if (!token) {
                alert('Bạn cần đăng nhập để mua hàng!')
                return
            }

            const payload = {
                id: item.product_id,
                name: item.product_name,
                price: item.product_price,
                thumbnail_url: item.product_thumbnail,
                quantity: 1,
            }

            localStorage.setItem('checkout_items', JSON.stringify([payload]))
            this.$router.push('/checkout')
        },

        openFeedback(billId) {
            if (!billId) {
                console.error("Không tìm thấy Bill ID trong dữ liệu đơn hàng");
                return;
            }
            this.selectedBillId = billId;
            this.showFeedbackModal = true;
        }
    },
}
</script>

<style scoped>
.content-box {
    background: #fff;
    border: 1px solid rgba(230,230,230,0.8);
    border-radius: 20px;
    padding: 28px;
    box-shadow:
        0 10px 30px rgba(91, 61, 245, 0.05);
    font-family: Segoe UI, sans-serif;
    font-weight: 400;
}

/* Tabs */
.order-tabs {
    list-style: none;
    padding: 0;
    margin: 0 0 28px 0;
    border-bottom: 1px solid #eee;
    gap: 10px;
}

.order-tabs li {
    padding: 12px 18px;
    cursor: pointer;
    color: #777;
    border-bottom: 2px solid transparent;
    font-weight: 500;
    transition: all .25s ease;
    position: relative;
}

.order-tabs li:hover {
    color: #111;
}

.order-tabs li.active {
    color: #5b3df5;
    font-weight: 600;
}

.order-tabs li.active::after {
    content: '';
    position: absolute;
    left: 0;
    bottom: -1px;
    width: 100%;
    height: 2px;
    border-radius: 999px;
    background: linear-gradient(
        90deg,
        #5b3df5,
        #7c3aed
    );
}

/* Order Card */
.order-wrapper {
    border-radius: 18px !important;
    border: 1px solid rgba(230,230,230,0.8) !important;
    transition: all .25s ease;
    background: #fff;
}

.order-wrapper:hover {
    transform: translateY(-2px);
    box-shadow:
        0 12px 30px rgba(91, 61, 245, 0.08);
}

/* Product Box */
.order-item {
    border-radius: 16px !important;
    border: 1px solid #f1f1f1 !important;
    background: #fafafa;
}

/* Thumbnail */
.category-thumb {
    width: 42px;
    height: 42px;
    object-fit: cover;
    border-radius: 10px;
    border: 1px solid #eee;
}

.image-rounded {
    width: 95px;
    height: 95px;
    object-fit: cover;
    border-radius: 14px;
    border: 1px solid #eee;
}

/* Status */
.order-status {
    font-size: 12px;
    font-weight: 700;
    padding: 8px 14px;
    border-radius: 999px;
    letter-spacing: .4px;
}

.order-status.paid {
    background: #e7f8ea;
    color: #1c9b45;
}

.order-status.status-direct {
    background: #fff2de;
    color: #d07b00;
}

.order-status.status-shipping {
    background: #e8f0ff;
    color: #3167d6;
}

/* Buttons */
.btn {
    border-radius: 12px !important;
    font-weight: 500;
    padding: 8px 14px;
    transition: all .25s ease;
}

.btn-outline-primary {
    border-color: #5b3df5 !important;
    color: #5b3df5 !important;
}

.btn-outline-primary:hover {
    background: #5b3df5 !important;
    color: #fff !important;
}

.btn-outline-secondary:hover {
    background: #111 !important;
    color: #fff !important;
}

/* Product name */
.order-item h6 {
    font-weight: 600;
    margin-bottom: 8px;
    color: #111;
}

/* Price */
.text-danger {
    color: #e53935 !important;
    font-size: 18px;
}
</style>