<template>
    <HeaderComponent />

    <div class="body container py-5">
        <div class="row g-5">
            <div class="col-md-6 text-center">
                <div class="product-image-wrap rounded">
                    <img
                        v-if="isValidThumbnail(product_detail.thumbnail_url)"
                        :src="product_detail.thumbnail_url"
                        class="img-fluid zoom-on-hover"
                        :alt="product_detail.name"
                        @error="product_detail.thumbnail_url = null"
                        style="max-height: 500px; object-fit: contain"
                    />
                    <img
                        v-else
                        src="@/assets/default_thumbnail.jpg"
                        class="img-fluid"
                        alt="Default Image"
                        style="max-height: 500px; object-fit: contain"
                    />
                </div>
            </div>

            <div class="col-md-6">
                <span class="badge bg-warning text-dark mb-2">Hot</span>
                <h3 class="fw-bold">{{ product_detail.name }}</h3>
                <p class="text-muted mb-1">trong {{ product_detail.category_title }}</p>

                <div class="mb-3">
                    <span style="font-size: 20px; font-weight: bold; color: #ffc107">
                        {{ Number(product_detail.score || 0).toFixed(2) }}
                    </span>
                    <span class="text-warning" style="color: #ffc107; margin: 0 5px">
                        {{ '★'.repeat(product_detail.score || 0)
                        }}{{ '☆'.repeat(5 - (product_detail.score || 0)) }}
                    </span>
                    <span
                        style="
                            font-size: 16px;
                            color: #6c757d;
                            font-weight: semi-bold;
                            margin-left: 5px;
                        "
                    >
                        | Đã bán {{ product_detail.total_sold || 0 }} sản phẩm
                    </span>
                </div>

                <div class="d-flex justify-content-between align-items-center mb-3">
                    <h4 class="fw-bold text-dark m-0">
                        {{ formatPrice(product_detail.price) }}
                    </h4>
                    <p class="text-success fw-semibold m-0">
                        Còn {{ product_detail.quantity || 0 }} sản phẩm
                    </p>
                </div>

                <div class="d-flex align-items-center mb-4">
                    <div class="input-group" style="width: 120px">
                        <button class="btn btn-outline-secondary" @click="minus">-</button>
                        <input
                            type="number"
                            class="form-control text-center no-spinner"
                            v-model.number="quantity"
                            @input="onInput"
                            :max="product_detail.quantity"
                            min="1"
                        />
                        <button class="btn btn-outline-secondary" @click="plus">+</button>
                    </div>
                </div>

                <div class="d-flex gap-3 mb-3 flex-wrap">
                    <button class="btn px-4 py-3 flex-fill btn-add-cart" @click="addToCartDetail">
                        <i class="bi bi-cart-plus me-2"></i> Thêm vào giỏ
                    </button>

                    <button class="btn px-4 py-3 flex-fill btn-buy-now" @click="buyNow">
                        Mua ngay
                    </button>
                </div>

                <ul class="list-unstyled small text-muted mt-4">
                    <li>🚚 Miễn phí giao hàng & hoàn hàng</li>
                    <li>📦 Vận chuyển đến tay trong 3-5 ngày làm việc</li>
                </ul>
            </div>
        </div>

        <div class="product-description mt-5">
            <ul class="nav nav-tabs" id="productTabs" role="tablist">
                <li class="nav-item" role="presentation">
                    <button
                        class="nav-link active"
                        id="desc-tab"
                        data-bs-toggle="tab"
                        data-bs-target="#desc"
                        type="button"
                        role="tab"
                    >
                        Mô tả
                    </button>
                </li>
                <li class="nav-item" role="presentation">
                    <button
                        class="nav-link"
                        id="review-tab"
                        data-bs-toggle="tab"
                        data-bs-target="#review"
                        type="button"
                        role="tab"
                    >
                        Đánh giá
                    </button>
                </li>
            </ul>

            <div class="tab-content p-4 border border-top-0 rounded-bottom" id="productTabsContent">
                <div class="tab-pane fade show active" id="desc" role="tabpanel">
                    <h5 class="fw-bold mb-3">{{ product_detail.name }}</h5>
                    <p>{{ product_detail.description }}</p>
                </div>
                <div class="tab-pane fade show" id="review" role="tabpanel">
                    <div class="row g-4">
                        <div class="col-md-4">
                            <div class="border p-4 rounded-3">
                                <h5 class="fw-bold mb-3">Đánh giá</h5>

                                <div class="text-center mb-3">
                                    <h1 class="fw-bold mb-0">
                                        {{ product_detail.score || 0 }}
                                        <span class="text-warning mb-0">★</span>
                                    </h1>

                                    <p class="text-muted small">
                                        {{ feedback_count || 0 }} lượt đánh giá
                                    </p>
                                </div>
                                <hr />
                                <div class="mt-3">
                                    <h6 class="fw-semibold">Đánh giá về sản phẩm</h6>
                                    <p class="text-muted small mb-3">
                                        Chia sẻ cảm nghĩ của bạn cho khách hàng khác
                                    </p>
                                </div>
                            </div>
                        </div>

                        <!-- Customer reviews -->
                        <div class="col-md-8">
                            <h5 class="fw-bold mb-3">
                                Khách hàng đánh giá ({{ feedback_count || 0 }})
                            </h5>
                            <div
                                v-for="fb in product_feedbacks"
                                :key="fb.id"
                                class="border-top pt-3"
                            >
                                <div class="d-flex justify-content-between align-items-start">
                                    <div class="d-flex align-items-center">
                                        <img
                                            :src="
                                                fb.user_avatar
                                                    ? fb.user_avatar
                                                    : 'https://ui-avatars.com/api/?name=' +
                                                      fb.user_name
                                            "
                                            alt="Avatar"
                                            class="rounded-circle me-3"
                                            width="48"
                                            height="48"
                                        />
                                        <div>
                                            <h6 class="mb-0 fw-semibold">{{ fb.user_name }}</h6>
                                            <small class="text-muted"
                                                >{{
                                                    new Date(fb.created_at).toLocaleDateString(
                                                        'vi-VN',
                                                    )
                                                }}
                                            </small>
                                        </div>
                                    </div>
                                    <div class="text-warning">
                                        {{ '★'.repeat(fb.score) }}{{ '☆'.repeat(5 - fb.score) }}
                                    </div>
                                </div>

                                <p class="mt-3 mb-0">
                                    {{ fb.comment }}
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Sản phẩm liên quan -->
    <div class="product-section">
        <div class="section-header d-flex justify-content-between align-items-center mb-4">
            <h4 class="fw-bold">Sản phẩm liên quan</h4>
        </div>
        <div class="row g-4">
            <div v-for="product in similar_products" :key="product.id" class="col-md-2_4 col-sm-6">
                <div
                    v-on:click="getDetail(product.id)"
                    class="product-card text-center p-3 rounded-3 position-relative"
                >
                    <span class="badge bg-warning text-dark position-absolute top-0 start-0 m-2"
                        >Hot</span
                    >
                    <img
                        v-if="product.thumbnail_url"
                        :src="product.thumbnail_url"
                        @error="product.thumbnail_url = null"
                        :alt="product.title || 'Small Appliances'"
                        class="img-fluid"
                    />
                    <img
                        v-else
                        src="@/assets/default_thumbnail.jpg"
                        alt="Default Image"
                        class="img-fluid"
                    />
                    <h6 class="fw-semibold product-name">{{ product.name }}</h6>
                    <p class="text-warning">
                        <span
                            >{{ '★'.repeat(product.score || 0)
                            }}{{ '☆'.repeat(5 - (product.score || 0)) }}</span
                        >
                    </p>

                    <p class="text-dark fw-bold mb-2">
                        {{
                            new Intl.NumberFormat('vi-VN', {
                                style: 'currency',
                                currency: 'VND',
                            }).format(product.price)
                        }}
                    </p>
                    <h6 class="fw-light product-name fst-italic text-muted">
                        {{ product.total_sold }} lượt mua
                    </h6>

                    <div class="product-actions">
                        <button
                            class="btn btn-add-cart w-100 mb-2"
                            @click.stop="addToCartFromSimilar(product)"
                        >
                            Thêm vào giỏ
                        </button>
                        <button class="btn btn-buy-now w-100">Mua ngay</button>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="showSeeMore" class="seemore-container">
            <p @click="listProduct()" class="btn-seemore">Xem thêm</p>
        </div>
    </div>

    <FooterComponent />
</template>

<script>
import axios from 'axios'
import HeaderComponent from '../components/HeaderComponent.vue'
import FooterComponent from '@/components/FooterComponent.vue'
import BannerComponent from '@/components/BannerComponent.vue'
import ProductComponent from '@/components/ProductComponent.vue'
import { apiHelper } from '@/helpers/axios'
import { mapStores } from 'pinia'
import { useCategoriesStore } from '@/stores/categories'
import { useCartStore } from '@/stores/cart'

export default {
    components: {
        HeaderComponent,
        FooterComponent,
        BannerComponent,
        ProductComponent,
    },

    props: {
        product: Object,
    },

    data() {
        return {
            product_detail: {},
            product_feedback: '',
            product_feedbacks: [],
            list_products: [],
            best_products: [],
            bgColors: ['#d8f3dc', '#ffe0ef', '#dce9f5', '#f8e7db'],
            similar_products: [],
            feedback_count: 0,
            quantity: 1,
            showSeeMore: false,
        }
    },

    created() {},
    mounted() {
        this.getDetail()
        this.listProduct()
        this.bestProducts()
        if (this.categoriesStore && this.categoriesStore.fetchListCategory) {
            this.categoriesStore.fetchListCategory()
        }
    },
    watch: {},
    computed: {
        ...mapStores(useCategoriesStore),
    },
    methods: {
        isValidThumbnail(url) {
            if (!url) return false
            const value = String(url).trim().toLowerCase()
            if (!value) return false
            if (value === 'null' || value === 'undefined') return false
            return true
        },
        formatPrice(value) {
            if (value === null || value === undefined || value === '') return '0 đ'
            return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.') + ' đ'
        },

        getDetail(id) {
            if (id == null) {
                id = this.$route.params.id
            }
            try {
                apiHelper
                    .get('/product-detail', {
                        params: {
                            product_id: id,
                        },
                    })
                    .then((res) => {
                        if (res.status == 200) {
                            this.product_detail = res.data.data.product || {}
                            this.product_feedbacks = res.data.data.product_feedbacks || []
                            this.feedback_count = res.data.data.feedback_count || 0
                            this.similar_products = res.data.data.similar_products || []

                            this.quantity = 1

                            window.scrollTo(0, 0)
                        }
                    })
                    .catch((err) => {
                        console.error(err)
                    })
            } catch (error) {
                console.log(error)
            }
        },

        listProduct() {
            try {
                apiHelper
                    .get('/list-product')
                    .then((res) => {
                        if (res.status == 200) {
                            this.list_products = res.data.data.list_products || []
                        }
                    })
                    .catch((error) => {
                        console.log(error)
                    })
            } catch (error) {
                console.log(error)
            }
        },

        bestProducts() {
            try {
                apiHelper
                    .get('/best-products')
                    .then((res) => {
                        if (res.status == 200) {
                            this.best_products = res.data.data.best_products || []
                        }
                    })
                    .catch((error) => {
                        console.log(error)
                    })
            } catch (error) {
                console.log(error)
            }
        },

        plus() {
            if (Number(this.quantity) < Number(this.product_detail.quantity)) {
                this.quantity = Number(this.quantity) + 1
            }
        },
        minus() {
            if (Number(this.quantity) > 1) {
                this.quantity = Number(this.quantity) - 1
            }
        },

        onInput(e) {
            if (this.quantity < 1) this.quantity = 1
            if (this.quantity > this.product_detail.quantity)
                this.quantity = this.product_detail.quantity
        },

        addToCartDetail() {
            const product = this.product_detail
            if (!product || !product.id) return

            const cartStore = useCartStore()
            const payload = {
                ...product,
                quantity: Number(this.quantity) || 1,
            }

            cartStore.addToCart(payload)

            alert('Sản phẩm đã được thêm vào giỏ hàng của bạn')
        },

        addToCartFromSimilar(product) {
            if (!product || !product.id) return

            const cartStore = useCartStore()
            cartStore.addToCart({
                ...product,
                quantity: 1,
            })

            alert('Sản phẩm đã được thêm vào giỏ hàng của bạn')
        },

        buyNow() {
            const token = sessionStorage.getItem('token')

            if (!token) {
                alert('Bạn cần đăng nhập để mua hàng, đăng nhập ngay')
                this.$router.push('/login')
                return
            }

            const product = this.product_detail
            if (!product || !product.id) {
                alert('Không tìm thấy thông tin sản phẩm')
                return
            }

            const payload = {
                ...product,
                quantity: 1,
            }

            localStorage.setItem('checkout_items', JSON.stringify([payload]))
            this.$router.push('/checkout')
        },
    },
}
</script>

<style scoped>
body {
    font-family: 'Outfit', Arial, sans-serif;
    background: #fff;
}

.container {
    max-width: 1200px;
    margin: auto;
    padding: 20px 20px 0 20px;
}

.product-description {
    max-width: 1200px;
    margin: auto;
}

.product-section {
    max-width: 1200px;
    margin: auto;
    padding: 0 20px 0 20px;
}

/* Toàn bộ text chuyển về màu tối */
* {
    color: #212529 !important;
}

/* Giữ màu vàng của sao */
.text-warning,
.text-warning * {
    color: #ffc107 !important;
}

/* Giữ màu nút Thêm vào giỏ và icon bên trong */
.btn-primary,
.btn-primary * {
    color: #fff !important;
    /* chữ và icon đều trắng */
}

/* Giữ màu nút Mua ngay */
.btn-outline-dark {
    color: #212529 !important;
}

/* Giữ màu xanh cho text-success */
.text-success {
    color: #198754 !important;
}

/* Giữ màu xanh dương cho text-primary */
.text-primary {
    color: #0d6efd !important;
}

/* Giữ màu xám cho text-muted */
.text-muted {
    color: #6c757d !important;
}

/* Giữ màu badge Hot */
.bg-warning {
    color: #212529 !important;
}

/* Nút thêm vào giỏ */
.btn-add-cart {
    background-color: #3449ca !important;
    color: #fff !important;
}

.btn-add-cart i {
    color: #fff !important;
}

/* Nút mua ngay */
.btn-buy-now {
    border: 2px solid #3449ca !important;
    color: #3449ca !important;
}

/* Hover nền xanh nhạt */
.btn-buy-now:hover {
    background-color: #e7e9fd !important;
    /* xanh siêu nhạt */
    color: #3449ca !important;
}

.zoom-on-hover {
    transition: transform 0.3s ease;
    /* mượt */
    cursor: pointer;
    /* optional, thấy tương tác */
    transform-origin: center;
}

.zoom-on-hover:hover {
    transform: scale(1.08);
}

.product-image-wrap {
    width: 100%;
    max-height: 500px;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
}

/* Ẩn nút tăng/giảm mặc định của input number trên mọi trình duyệt */
input[type='number'] {
    -moz-appearance: textfield;
    /* Firefox */
}

input[type='number']::-webkit-inner-spin-button,
input[type='number']::-webkit-outer-spin-button {
    -webkit-appearance: none;
    /* Chrome, Safari, Edge */
    margin: 0;
}

.product-section {
    position: relative;
    z-index: 1;
    overflow: visible;
}

.container {
    max-width: 1200px;
    margin: auto;
    padding: 20px 20px 0 20px;
}

.container-custom {
    max-width: 1250px;
    margin: 0 auto;
    padding: 0 50px;
}

.menu-row {
    border-bottom: 1px solid #eaeaea;
    background: #fff;
}

.col-md-2_4 {
    flex: 0 0 20%;
    max-width: 20%;
    position: relative;
    padding-bottom: 120px;
}

@media (max-width: 991px) {
    .col-md-2_4 {
        flex: 0 0 33.3333%;
        max-width: 33.3333%;
    }
}

.product-card {
    position: relative;
    background: #fff;
    border: 1px solid #eee;
    border-radius: 8px;
    overflow: visible;
    transition: all 0.3s ease;
    z-index: 1;
    padding: 15px;
    text-align: center;
}

.product-card img {
    width: 100%;
    height: 160px;
    object-fit: contain;
    transition: all 0.3s ease;
}

.product-card .product-name {
    font-size: 15px;
    font-weight: 500;
    height: 35px;
    overflow: hidden;
    margin: 10px 0 6px;
}

.product-card .product-price {
    color: #333;
    font-weight: 700;
    margin-bottom: 12px;
}

.product-card .product-actions {
    position: absolute;
    bottom: -110px;
    /* ↓ thấp hơn 1 chút để không che giá */
    left: 0;
    width: 100%;
    background: #fff;
    padding: 16px 15px 18px;
    box-shadow: 0 18px 35px rgba(0, 0, 0, 0.15);
    border-radius: 0 0 8px 8px;

    opacity: 0;
    visibility: hidden;
    transform: translateY(10px);
    transition: all 0.3s ease;
}

/* nút */
.product-card .product-actions .btn {
    display: block;
    width: 100%;
    margin-bottom: 10px;
    border-radius: 4px;
    font-size: 14px;
    padding: 10px;
}

/* hover */
.product-card:hover {
    transform: translateY(-6px);
    box-shadow: 0 14px 30px rgba(0, 0, 0, 0.25);
    z-index: 5;
}

.product-card:hover .product-actions {
    opacity: 1;
    visibility: visible;
    transform: translateY(0);
}

@media (prefers-color-scheme: dark) {
    html,
    body,
    h1,
    h2,
    h3,
    h4,
    h5,
    h6,
    p,
    span,
    a,
    li,
    label,
    input,
    select,
    button,
    small,
    .fw-bold,
    .fw-semibold,
    .text-dark,
    .text-muted,
    .nav-link,
    .dropdown-item,
    .footer a,
    .footer p {
        color: #111 !important;
    }

    .text-muted {
        color: #555 !important;
    }

    svg path {
        fill: #111 !important;
        stroke: #111 !important;
    }

    .btn-outline-dark {
        color: #111 !important;
        border-color: #111 !important;
    }

    .btn-outline-dark:hover {
        background-color: #111 !important;
        color: #fff !important;
    }

    .btn-primary {
        color: #fff !important;
    }

    .badge.bg-warning {
        color: #000 !important;
    }
}

.seemore-container {
    text-align: center;
    margin-top: 100px;
}

.btn-seemore {
    display: inline-block;
    color: var(--primary);
    border: 1px solid var(--primary);
    border-radius: 20px;
    padding: 6px 16px;
    font-size: 14px;
    transition: all 0.2s ease;
    cursor: pointer;
}

.btn-seemore:hover {
    font-weight: bolder;
}

.sort-container {
    display: flex;
    align-items: center;
    justify-content: flex-end;
}

.sort-container label {
    font-size: 14px;
    color: #555;
}

.sort-container select {
    border-radius: 6px;
    border: 1px solid #ddd;
    padding: 4px 8px;
    font-size: 14px;
    cursor: pointer;
}

.sort-container select:focus {
    outline: none;
    border-color: var(--primary);
    box-shadow: 0 0 3px var(--primary);
}

/* Nút thêm vào giỏ */
.product-actions .btn-add-cart,
.btn-add-cart {
    background-color: #3449ca !important;
    color: #fff !important;
    border: none !important;
}

.product-actions .btn-add-cart i,
.btn-add-cart i {
    color: #fff !important;
}

/* Nút mua ngay */
.product-actions .btn-buy-now,
.btn-buy-now {
    border: 2px solid #3449ca !important;
    color: #3449ca !important;
    background: #fff !important;
}

/* Hover của nút mua ngay */
.product-actions .btn-buy-now:hover,
.btn-buy-now:hover {
    background-color: #e7e9fd !important;
    /* xanh nhạt */
    color: #3449ca !important;
}
</style>
