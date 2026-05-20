<template>
    <HeaderComponent />

    <div class="body container py-5">
        <div class="container">
            <h1 class="cart-title">GIỎ HÀNG</h1>

            <div class="row">
                <div class="col-md-8">
                    <div class="row product-row align-items-center" v-for="(item, index) in products" :key="item.id">
                        <div class="col-auto d-flex align-items-center">
                            <input type="checkbox" v-model="item.selected" class="product-checkbox" />
                        </div>

                        <div class="col-3 col-md-2">
                            <img :src="item.thumbnail_url || defaultThumbnail" class="product-img" />
                        </div>

                        <div class="col-6 col-md-5">
                            <div class="product-name">{{ item.name }}</div>
                            <div class="product-price">{{ formatPrice(item.price) }}</div>

                            <div class="d-flex align-items-center mt-2">
                                <button class="qty-btn" @click="decrease(index)">-</button>
                                <div class="qty-number">{{ item.quantity }}</div>
                                <button class="qty-btn" @click="increase(index)">+</button>
                            </div>
                        </div>

                        <div class="col-md-2 text-md-right">
                            <strong class="product-total">
                                {{ formatPrice(item.price * item.quantity) }}
                            </strong>
                        </div>

                        <div class="col-auto text-right">
                            <span class="trash-icon" @click="removeItem(index)">&#128465;</span>
                        </div>
                    </div>
                </div>

                <div class="col-md-4 mt-4 mt-md-2">
                    <div class="cart-box">
                        <h5 class="cart-box-title text-dark mb-3 text-center">
                            Tổng số giỏ hàng
                        </h5>

                        <div class="d-flex justify-content-between">
                            <span class="text-dark">Thêm mã giảm giá</span>
                            <span class="text-dark">⌄</span>
                        </div>

                        <hr />

                        <div class="d-flex justify-content-between">
                            <strong class="text-dark">Tổng ước tính</strong>
                            <strong class="text-dark">{{ formatPrice(totalPrice) }}</strong>
                        </div>

                        <button class="checkout-btn" @click="goToCheckout">
                            Tiến hành thanh toán
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <FooterComponent />
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { useRouter } from "vue-router";
import HeaderComponent from "../components/HeaderComponent.vue";
import FooterComponent from "@/components/FooterComponent.vue";
import { useCartStore } from "@/stores/cart";
import defaultThumbnail from "../assets/default_thumbnail.jpg";

const router = useRouter();
const cartStore = useCartStore();
const products = ref([]);

onMounted(() => {
    const storedCart = JSON.parse(localStorage.getItem("cart") || "[]");
    if (storedCart.length) {
        products.value = storedCart.map((item) => ({
            ...item,
            quantity: item.quantity || 1,
            selected: false
        }));
        cartStore.items = [...products.value];
    } else if (cartStore.items.length) {
        products.value = cartStore.items.map((item) => ({
            ...item,
            quantity: item.quantity || 1,
            selected: false
        }));
        localStorage.setItem("cart", JSON.stringify(products.value));
    }
});

const increase = (index) => {
    products.value[index].quantity++;
    cartStore.items[index].quantity = products.value[index].quantity;
    localStorage.setItem("cart", JSON.stringify(cartStore.items));
};

const decrease = (index) => {
    if (products.value[index].quantity > 1) {
        products.value[index].quantity--;
        cartStore.items[index].quantity = products.value[index].quantity;
        localStorage.setItem("cart", JSON.stringify(cartStore.items));
    }
};

const removeItem = (index) => {
    products.value.splice(index, 1);
    cartStore.items.splice(index, 1);
    localStorage.setItem("cart", JSON.stringify(cartStore.items));
};

const totalPrice = computed(() =>
    products.value
        .filter((item) => item.selected)
        .reduce((sum, item) => sum + item.price * item.quantity, 0)
);

const formatPrice = (value) => {
    if (!value) return "0 ₫";
    return new Intl.NumberFormat("vi-VN", {
        style: "currency",
        currency: "VND"
    }).format(value);
};

const goToCheckout = () => {
    const token = sessionStorage.getItem("token");
    if (!token) {
        alert("Bạn cần đăng nhập để mua hàng, đăng nhập ngay");
        router.push("/login");
        return;
    }
    const selectedProducts = products.value.filter(item => item.selected);

    if (selectedProducts.length === 0) {
        alert("Bạn vui lòng chọn mặt hàng");
        return;
    }

    localStorage.setItem("checkout_items", JSON.stringify(selectedProducts));

    router.push("/checkout");
};
</script>

<style scoped>
body {
    font-family: Segoe UI, sans-serif;
    background: #fff;
    color: #111;
}

.container {
    max-width: 1180px;
}

/* Title */
.cart-title {
    font-size: 32px;
    font-weight: 800;
    margin-bottom: 50px;
    text-align: center;
    color: #151515;
    letter-spacing: -1px;
}

/* Product Row */
.product-row {
    padding: 24px;
    margin-bottom: 18px;

    border:
        1px solid rgba(230,230,230,0.8);

    border-radius: 24px;

    background:
        rgba(255,255,255,0.82);

    backdrop-filter: blur(12px);

    box-shadow:
        0 10px 30px rgba(91, 61, 245, 0.05);

    transition: all .28s ease;
}

.product-row:hover {
    transform: translateY(-3px);

    box-shadow:
        0 18px 40px rgba(91, 61, 245, 0.08);
}

/* Checkbox */
.product-checkbox {
    width: 22px;
    height: 22px;
    cursor: pointer;
    accent-color: #5b3df5;
}

/* Product image */
.product-img {
    width: 90px;
    height: 90px;
    object-fit: cover;

    border-radius: 18px;

    border:
        1px solid #eee;

    background: #fff;

    padding: 6px;
}

/* Product name */
.product-name {
    font-size: 18px;
    font-weight: 700;
    color: #151515;
    margin-bottom: 8px;
}

/* Product price */
.product-price {
    color: #777;
    font-size: 15px;
}

/* Total */
.product-total {
    color: #e53935;
    font-size: 20px;
    font-weight: 700;
}

/* Quantity */
.qty-btn {
    width: 36px;
    height: 36px;

    border: none;

    border-radius: 12px;

    background:
        rgba(91, 61, 245, 0.08);

    color: #5b3df5;

    font-weight: 700;

    transition: all .25s ease;
}

.qty-btn:hover {
    background:
        linear-gradient(
            135deg,
            #5b3df5,
            #7c3aed
        );

    color: white;
}

.qty-number {
    min-width: 30px;
    text-align: center;

    margin: 0 10px;

    font-weight: 700;
    color: #151515;
}

/* Trash */
.trash-icon {
    font-size: 24px;
    cursor: pointer;

    color: #d62839;

    transition: all .25s ease;
}

.trash-icon:hover {
    transform: scale(1.15);
    color: #b00020;
}

/* Cart Summary Box */
.cart-box {
    position: sticky;
    top: 120px;

    background:
        rgba(255,255,255,0.85);

    backdrop-filter: blur(14px);

    padding: 30px;

    border-radius: 24px;

    border:
        1px solid rgba(230,230,230,0.8);

    box-shadow:
        0 14px 35px rgba(91, 61, 245, 0.08);
}

/* Box title */
.cart-box-title {
    font-size: 24px;
    font-weight: 700;
    color: #151515;
}

/* Text */
.text-dark {
    color: #151515 !important;
}

/* Checkout button */
.checkout-btn {
    width: 100%;

    margin-top: 28px;

    border: none;

    border-radius: 18px;

    padding: 16px 0;

    font-size: 16px;
    font-weight: 600;

    color: white;

    background:
        linear-gradient(
            135deg,
            #5b3df5,
            #7c3aed
        );

    box-shadow:
        0 12px 30px rgba(91, 61, 245, 0.22);

    transition: all .28s ease;
}

.checkout-btn:hover {
    transform: translateY(-2px);

    box-shadow:
        0 18px 40px rgba(91, 61, 245, 0.3);
}

/* HR */
hr {
    border-color: rgba(0,0,0,0.08);
}

/* Responsive */
@media (max-width: 768px) {
    .cart-title {
        font-size: 34px;
    }

    .product-row {
        padding: 18px;
    }

    .cart-box {
        margin-top: 20px;
        position: static;
    }

    .product-img {
        width: 75px;
        height: 75px;
    }
}
</style>