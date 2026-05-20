<template>
    <HeaderComponent />
    <BannerComponent />
    <div class="body">
        <!-- Sản phẩm bán chạy -->
        <div class="best-seller container mt-5">
            <div class="section-header d-flex justify-content-between align-items-center mb-4">
                <h4 class="fw-bold">Sản phẩm bán chạy</h4>
                <router-link to="/product" class="text-muted">
                    Tất cả sản phẩm
                </router-link>
            </div>

            <div class="row g-4">
                <div v-for="(product, index) in best_products" :key="product.id" class="col-md-3 col-sm-6">
                    <div class="best-seller-item" :style="{ backgroundColor: bgColors[index % bgColors.length] }">
                        <div class="content">
                            <h5 class="clamp-2">{{ product.name }}</h5>
                            <p>
                                <span v-for="i in product.score" :key="i">★</span>
                            </p>
                            <!-- <p class="small-desc">{{ product.description }}</p> -->
                            <div class="best-seller-description">
                                <p class="clamp-2">{{ product.description }}</p>
                            </div>
                            <a href="#">Mua ngay</a>
                        </div>
                        <div class="best-product-thumbnail">
                            <img v-if="product.thumbnail_url" :src="product.thumbnail_url"
                                style="object-fit: contain; border-radius: 2%" @error="product.thumbnail_url = null"
                                :alt="product.title || 'Small Appliances'" />
                            <img v-else src="@/assets/default_thumbnail.jpg" alt="Default Image" />
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Sản phẩm -->
        <div class="product-section container mt-5">
            <div class="section-header d-flex justify-content-between align-items-center mb-4">
                <h4 class="fw-bold">Sản phẩm</h4>
                <router-link to="/product" class="text-muted">
                    Tất cả sản phẩm
                </router-link>
            </div>
            <ProductComponent :showSeeMore="false" :showSortType="false" :showTitle="false" />
        </div>

        <!-- Categories -->
        <div class="categories container mt-5 mb-5">
            <h4 class="fw-bold mb-4">Danh mục</h4>
            <div class="row text-center g-4 justify-content-center">
                <div v-for="category in categoriesStore.listCategory" :key="category.id"
                    class="col-lg-2 col-md-3 col-sm-4 col-6">
                    <div class="category-item">
                        <img v-if="category.thumbnail_url" :src="category.thumbnail_url"
                            @error="category.thumbnail_url = null" :alt="category.title"
                            class="rounded-circle img-fluid mb-2" />
                        <img v-else src="@/assets/default_thumbnail.jpg" alt="default image"
                            class="rounded-circle img-fluid mb-2" />

                        <p style="font-weight: bold">{{ category.title }}</p>
                    </div>
                </div>
                <!-- Giữ nguyên category cuối -->
                <div class="col-lg-2 col-md-3 col-sm-4 col-6">
                    <div class="category-item">
                        <img src="../assets/chamcham.png" alt="Headphones" class="rounded-circle img-fluid mb-2" />
                        <p style="font-weight: bold">More</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <FooterComponent />
</template>

<script setup>
import axios from 'axios'
import HeaderComponent from '../components/HeaderComponent.vue'
import FooterComponent from '@/components/FooterComponent.vue'
import BannerComponent from '@/components/BannerComponent.vue'
import { apiHelper } from '@/helpers/axios'
import { mapStores } from 'pinia'
import { useCategoriesStore } from '@/stores/categories'
import ProductComponent from '@/components/ProductComponent.vue'
</script>

<script>
export default {
    props: {
        product: Object,
    },

    data() {
        return {
            list_products: [],
            best_products: [],
            bgColors: ['#d8f3dc', '#ffe0ef', '#dce9f5', '#f8e7db'], // 4 màu nền xoay vòng,
            // categories: [],
            token: sessionStorage.getItem('token'),
        }
    },
    created() { },
    mounted() {
        this.listProduct()
        this.bestProducts()
        this.categoriesStore.fetchListCategory()
        this.userInfor()
    },
    watch: {},
    computed: {
        ...mapStores(useCategoriesStore),
    },
    methods: {
        /*************  ✨ Windsurf Command 🌟  *************/
        /**
         * List all categories
         * @return {Promise<void>}
         */
        listProduct() {
            try {
                /**
                 * Make a GET request to the API to list all products
                 * @return {Promise<AxiosResponse>}
                 */
                apiHelper
                    .get('/list-product')
                    .then((res) => {
                        // console.log(res);
                        if (res.status == 200) {
                            /**
                             * Set the products data to the component's data
                             * @param {Object[]} products - The products data
                             */
                            this.list_products = res.data.data.list_products
                            // console.log(this.list_products);
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
                /**
                 * Make a GET request to the API to list 4 best products
                 * @return {Promise<AxiosResponse>}
                 */
                apiHelper
                    .get('/best-products')
                    .then((res) => {
                        // console.log(res);
                        if (res.status == 200) {
                            /**
                             * Set the products data to the component's data
                             * @param {Object[]} products - The best 4 products data
                             */
                            this.best_products = res.data.data.best_products
                            // console.log(this.best_products);
                        }
                    })
                    .catch((error) => {
                        console.log(error)
                    })
            } catch (error) {
                console.log(error)
            }
        },
        userInfor() {
            if (sessionStorage.getItem('token') !== null) {
                try {
                    apiHelper
                        .get('/user-infor', {
                            headers: {
                                Authorization: `Bearer ${this.token}`
                            }
                        })
                        .then((res) => {
                            if (res.status == 200) {
                                localStorage.setItem("auth", JSON.stringify(res.data.data));
                            }
                        })
                        .catch((error) => {
                            console.log(error)
                        })
                } catch (error) {
                    console.log(error)
                }
            }
        }
    },
}
</script>

<style scoped>
:root {
    --primary: #5b3df5;
    --primary-light: #ede9ff;
    --primary-soft: #f5f2ff;
    --text-dark: #151515;
    --text-gray: #777;
}

body {
    background: linear-gradient(
        to bottom,
        #f5f2ff,
        #ffffff 25%
    );
}
.section-header h4 {
    font-size: 30px;
    font-weight: 700;
    color: #151515;
}
.header-top {
    background-image: url('../assets/campain-bar-1.jpg');
    text-align: center;
    padding: 13px 0;
    font-size: 13px;
    font-weight: 600;
}

.header-top a {
    text-decoration: underline;
    color: #222;
    font-weight: 700;
}

.main-header {
    background: #fff;
    padding: 20px 0;
}

.container-custom {
    max-width: 1250px;
    margin: 0 auto;
    padding: 0 50px;
}

.navbar-brand {
    font-weight: 800;
    font-size: 28px;
    display: flex;
    align-items: center;
    gap: 8px;
}

.navbar-brand small {
    font-size: 12px;
    font-weight: 500;
    color: #666;
}

.search-wrap {
    display: flex;
    align-items: center;
    gap: 8px;
    width: 58%;
}

.search-box {
    flex: 1;
    display: flex;
    align-items: center;
    background: rgba(255,255,255,0.9);
    border-radius: 18px;
    border: 1px solid rgba(230,230,230,0.8);
    backdrop-filter: blur(10px);
    box-shadow:
        0 10px 25px rgba(91, 61, 245, 0.08);
}

.search-box input {
    height: 50px;
    border: 1px solid #eaeaea;
    outline: none;
    flex: 1;
    padding: 10px 14px;
    font-size: 15px;
    border-radius: 6px;
    background-color: #fff;
    box-shadow:
        0 2px 5px rgba(0, 0, 0, 0.05),
        0 4px 8px rgba(0, 0, 0, 0.04),
        2px 0 4px rgba(0, 0, 0, 0.025),
        -2px 0 4px rgba(0, 0, 0, 0.025);
    transition: all 0.2s ease;
}

.search-box input:focus {
    border-color: #007bff;
    box-shadow:
        0 4px 10px rgba(0, 123, 255, 0.15),
        0 6px 15px rgba(0, 123, 255, 0.1),
        2px 0 6px rgba(0, 123, 255, 0.08),
        -2px 0 6px rgba(0, 123, 255, 0.08);
}

.btn-search {
    width: 40px;
    height: 40px;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: var(--primary);
    color: #fff;
    border: none;
    transition: 0.2s;
}

.btn-search:hover {
    opacity: 0.9;
}

/* shop-category */
.menu-row {
    border-bottom: 1px solid #eaeaea;
    background: #fff;
}

.menu-row .container-custom {
    display: flex;
    align-items: center;
    gap: 24px;
    position: relative;
    padding: 0 20px;
}

/* Shop Category (Giới thiệu) */
.shop-category {
    height: 60px;
    width: 220px;
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 0 16px;
    border-radius: 6px;
    background: #fff;
    border: 1px solid #eaeaea;
    transition: all 0.3s ease;
    color: #111;
    text-decoration: none;
    position: relative;
    margin-left: 30px;
}

/* Giữ hover nhẹ, không gạch chân, không đổi màu */
.shop-category:hover {
    box-shadow: 0 6px 14px rgba(0, 0, 0, 0.1);
    transform: translateY(-2px);
    color: black;
}

/* Bỏ hoàn toàn hiệu ứng gạch chân */
.shop-category::after {
    display: none;
}

/* MENU CHÍNH */
.menu-nav {
    flex: 1;
    position: relative;
}

.menu-nav .nav {
    display: flex;
    align-items: center;
    gap: 25px;
    list-style: none;
    margin: 0 0 0 20px;
    padding: 0;
}

.menu-nav .nav-item {
    position: relative;
}

/* Menu cha – chữ đậm hơn, không đổi màu khi hover */
.menu-nav .nav-link {
    position: relative;
    color: #111;
    font-weight: 600;
    /* Đậm hơn */
    text-decoration: none;
    padding: 10px 0;
    display: inline-block;
    transition: color 0.3s ease;
}

/* Giữ hiệu ứng gạch chân động cho menu cha */
.menu-nav .nav-link::after {
    content: '';
    position: absolute;
    left: 0;
    bottom: 0;
    height: 2px;
    width: 0;
    background-color: black;
    transition: width 0.3s ease;
}

/* Hover menu cha: chỉ hiện gạch chân, không đổi màu chữ */
.menu-nav .nav-link:hover::after {
    width: 100%;
}

/* Dropdown menu */
.dropdown-menu {
    display: none;
    position: absolute;
    top: 120%;
    left: 0;
    min-width: 220px;
    background: #fff;
    border: 1px solid #eaeaea;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
    border-radius: 6px;
    padding: 10px 0;
    z-index: 50;
    opacity: 0;
    visibility: hidden;
    transition: all 0.3s ease;
}

/* Hiện dropdown khi hover */
.nav-item.has-dropdown:hover>.dropdown-menu {
    display: block;
    opacity: 1;
    visibility: visible;
    top: 100%;
}

.dropdown-menu li {
    list-style: none;
}

/* Danh mục con – không đổi màu khi hover, chỉ gạch chân chữ */
.dropdown-menu li a {
    display: inline-block;
    padding: 10px 16px;
    color: #111;
    font-weight: 500;
    text-decoration: none;
    position: relative;
    transition: all 0.3s ease;
}

.dropdown-menu li a::after {
    content: '';
    position: absolute;
    left: 0;
    bottom: 4px;
    height: 2px;
    width: 0;
    background-color: black;
    transition: width 0.3s ease;
}

.dropdown-menu li a:hover::after {
    width: 100%;
}

/* Tài khoản + Giỏ hàng */
.header-actions {
    display: flex;
    align-items: center;
    gap: 22px;
    position: relative;
}

.header-actions a,
.header-actions label {
    color: #111;
    font-weight: 700;
    cursor: pointer;
    text-decoration: none;
}

.header-actions i {
    font-size: 20px;
    margin-right: 6px;
    font-weight: 700;
}

/* --- Giỏ hàng --- */
.cart-badge {
    position: relative;
    display: inline-block;
}

.cart-badge .badge {
    position: absolute;
    top: -6px;
    right: -10px;
    background: #ff4d4d;
    color: #fff;
    font-size: 12px;
    padding: 4px 7px;
    border-radius: 50%;
}

.account-dropdown {
    position: relative;
}

#toggle-account {
    display: none;
}

.dropdown-menu-account {
    display: none;
    position: absolute;
    top: 130%;
    left: 0;
    background: #fff;
    border: 1px solid #eaeaea;
    border-radius: 6px;
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
    padding: 10px 0;
    min-width: 180px;
    z-index: 50;
}

.dropdown-menu-account a {
    display: flex;
    align-items: center;
    justify-content: space-between;
    /* đẩy icon sang phải */
    padding: 10px 16px;
    color: #111;
    font-weight: 500;
    text-decoration: none;
    transition: background 0.2s ease;
}

.dropdown-menu-account a svg {
    width: 18px;
    height: 18px;
    fill: #555;
    flex-shrink: 0;
    margin-left: 8px;
}

.dropdown-menu-account a:hover svg {
    fill: #000;
}

.dropdown-menu-account a:hover {
    background: #f9f9f9;
}

#toggle-account:checked~.dropdown-menu-account {
    display: block;
}

.dropdown-menu-account::before {
    content: '';
    position: absolute;
    top: -8px;
    left: 25px;
    border-width: 0 8px 8px 8px;
    border-style: solid;
    border-color: transparent transparent #fff transparent;
}

/* Banner */
.banner-section {
    width: 100%;
    margin: 0 auto;
    padding: 0;
}

.slider {
    position: relative;
    width: 100%;
    height: 480px;
    overflow: hidden;
    border-radius: 0;
    box-shadow: none;
    background-color: #f5f5f5;
}

.slides {
    display: flex;
    width: 100%;
    transition: transform 0.6s ease-in-out;
}

.slides img {
    width: 100%;
    height: 480px;
    object-fit: cover;
}

.slider-dots {
    position: absolute;
    bottom: 30px;
    left: 40px;
    display: flex;
    gap: 10px;
    z-index: 10;
}

.slider-dots span {
    display: block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background-color: rgba(255, 255, 255, 0.6);
    transition: all 0.3s;
    cursor: pointer;
}

.slider-dots span.active {
    background-color: #fff;
    transform: scale(1.2);
}

.slider-controls {
    position: absolute;
    bottom: 25px;
    right: 40px;
    display: flex;
    gap: 12px;
    z-index: 10;
}

.slider-controls button {
    background: rgba(255, 255, 255, 0.8);
    border: none;
    width: 45px;
    height: 45px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 22px;
    cursor: pointer;
    transition: all 0.3s;
}

.slider-controls button:hover {
    background: #fff;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
}

/* Body */
/* sản phẩm bán chạy */
.best-seller-item {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    border-radius: 24px;
    overflow: hidden;
    padding: 24px;
    background: rgba(255,255,255,0.75);
    backdrop-filter: blur(10px);
    border: 1px solid rgba(255,255,255,0.4);
    box-shadow:
        0 10px 30px rgba(91, 61, 245, 0.08);
    transition: all 0.3s ease;
    height: 100%;
}
.best-seller-item:hover {
    transform: translateY(-6px);

    box-shadow:
        0 18px 40px rgba(91, 61, 245, 0.18);
}
.best-seller-item img {
    width: 100%;
    height: 200px;
    object-fit: cover;
    border-radius: 6px;
    margin-bottom: 8px;
}

.best-seller-item h5 {
    font-size: 16px;
    font-weight: 700;
    margin: 0 0 4px 0;
    line-height: 1.25;
    height: calc(1.25em * 2);
    /* giữ max 2 dòng nếu tên dài */
    overflow: hidden;
}

.best-seller-item .small-desc {
    font-size: 14px;
    line-height: 1.25;
    height: calc(1.25em * 2);
    /* chính xác 2 dòng */
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    /* Safari/Chrome */
    -webkit-box-orient: vertical;
    text-overflow: ellipsis;
    word-break: break-word;
    margin: 0 0 8px 0;
}

.best-seller-item a {
    display: block;
    text-align: center;
    background: #5b3df5;
    color: white;
    padding: 12px 0;
    border-radius: 12px;
    text-decoration: none;
    font-weight: 600;

    transition: all 0.25s ease;
    box-shadow:
        0 8px 20px rgba(91, 61, 245, 0.2);
}
.best-seller-item a:hover {
    background: #4c2ff0;
    transform: translateY(-2px);
}

.best-product-thumbnail img {
    transition: transform 0.3s ease;
    /* mượt */
}

.best-product-thumbnail:hover img {
    transform: scale(1.1);
    /* phóng to 10% */
}


/* sản phẩm + cate */
.section-header a {
    color: #5b3df5 !important;
    font-weight: 600;
    text-decoration: none;
}
.section-header a:hover {
    opacity: 0.8;
}
.col-md-2_4 {
    flex: 0 0 20%;
    max-width: 20%;
    position: relative;
}

.product-section {
    position: relative;
    z-index: 1;
    overflow: visible;
}

.product-card {
    position: relative;
    background: rgba(255,255,255,0.9);
    border-radius: 24px;
    border: 1px solid rgba(230,230,230,0.8);
    backdrop-filter: blur(10px);
    transition: all 0.3s ease;
    z-index: 1;
    padding: 20px;
    text-align: center;
    overflow: hidden;
    box-shadow:
        0 10px 25px rgba(91, 61, 245, 0.06);
}
.product-card:hover {
    transform: translateY(-8px);

    box-shadow:
        0 20px 40px rgba(91, 61, 245, 0.18);
}
/* hình ảnh */
.product-card img {
    width: 100%;
    height: 160px;
    object-fit: contain;
    transition: all 0.3s ease;
}

/* tên sản phẩm */
.product-card .product-name {
    font-size: 15px;
    font-weight: 500;
    height: 35px;
    overflow: hidden;
    margin: 10px 0 6px;
}

/* giá */
.product-card .product-price {
    color: #333;
    font-weight: 700;
    margin-bottom: 12px;
}

/* phần chứa nút */
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

/* responsive */
@media (max-width: 991px) {
    .col-md-2_4 {
        flex: 0 0 33.3333%;
        max-width: 33.3333%;
    }
}

/* ==== CATEGORIES ==== */
.category-item {
    background: white;
    border-radius: 24px;
    padding: 24px 16px;
    transition: all 0.3s ease;
    box-shadow:
        0 8px 24px rgba(91, 61, 245, 0.06);
}
.category-item:hover {
    transform: translateY(-6px);
    box-shadow:
        0 16px 32px rgba(91, 61, 245, 0.14);
}
.category-item img {
    width: 170px;
    /* tăng từ 140px lên 170px */
    height: 170px;
    object-fit: cover;
    border: 4px solid #f2f2f2;
    /* nhẹ nhàng cho viền sáng */
    transition: filter 0.3s ease;
}

.category-item:hover img {
    filter: brightness(1.05);
    /* chỉ làm sáng nhẹ khi hover */
}

.category-item p {
    font-weight: 500;
    margin-top: 6px;
    color: #333;
    font-size: 15px;
}

/* Footer */
.footer {
    background: #fff;
    color: #222;
    font-family: 'Inter', sans-serif;
    border-top: 1px solid #eaeaea;
}

.container {
    max-width: 1200px;
    margin: auto;
    padding: 20px 20px 0 20px;
}

/* === TOP === */
.footer-top {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    border-bottom: 1px solid #eee;
    padding-top: 30px;
    margin-bottom: 40px;
}

.footer-col {
    flex: 1;
    min-width: 240px;
    margin-bottom: 20px;
}

.footer-col h4,
.footer-col h3 {
    font-weight: 600;
    margin-bottom: 8px;
}

.footer-col p,
.footer-col a {
    color: #555;
    font-size: 14px;
    line-height: 1.6;
}

/* === MIDDLE === */
.footer-middle {
    display: grid;
    grid-template-columns: 1.3fr 2fr;
    gap: 40px;
    align-items: flex-start;
    padding-bottom: 10px;
}

.footer-logo {
    font-weight: 700;
    font-size: 26px;
    margin-bottom: 6px;
}

.footer-logo span {
    color: #ff4d00;
}

.footer-brand .socials a {
    margin-right: 12px;
    font-size: 18px;
    color: #333;
    transition: color 0.3s;
}

.footer-brand .socials a:hover {
    color: #007bff;
}

.footer-links {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 30px;
}

.footer-list h4 {
    margin-bottom: 10px;
    font-size: 15px;
    font-weight: 600;
}

.footer-list ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.footer-list li {
    margin-bottom: 6px;
}

.footer-list a {
    color: #555;
    text-decoration: none;
    font-size: 14px;
    transition: color 0.3s;
}

.footer-list a:hover {
    color: #007bff;
}

/* === BOTTOM === */
.footer-bottom {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    border-top: 1px solid #eee;
    padding-top: 20px;
    align-items: center;
}

.footer-policy {
    list-style: none;
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    padding: 0;
    margin: 0;
}

.footer-policy a {
    color: #666;
    font-size: 14px;
    text-decoration: none;
}

.footer-payments img {
    height: 22px;
    margin-left: 12px;
    vertical-align: middle;
}

.footer-copy {
    text-align: center;
    font-size: 13px;
    color: #888;
    margin-top: 20px;
    padding-bottom: 20px;
}

@media (max-width: 991px) {
    .footer-middle {
        grid-template-columns: 1fr;
    }

    .footer-links {
        grid-template-columns: repeat(2, 1fr);
    }

    .footer-bottom {
        flex-direction: column;
        align-items: flex-start;
        gap: 15px;
    }
}

@media (max-width: 576px) {
    .footer-links {
        grid-template-columns: 1fr;
    }

    .footer-top {
        flex-direction: column;
    }

    .footer-copy {
        font-size: 12px;
    }
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

/* Giới hạn 2 dòng, hiển thị ... nếu quá dài */
/* Giới hạn 2 dòng thực sự, cắt hẳn phần dư */
.clamp-2 {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    /* 2 dòng */
    -webkit-box-orient: vertical;
    overflow: hidden;
    /* ẩn phần dư */
    text-overflow: ellipsis;
    /* ... nếu quá dài */
    word-break: break-word;
    /* tránh tràn chữ dài */
}

/* Thêm khoảng cách giữa content và ảnh */
.best-seller-item img {
    margin-top: 8px;
    /* giảm từ 12px xuống 8px để gần nội dung hơn */
}

/* Giảm khoảng cách trên và dưới description */
.best-seller-item .small-desc {
    margin: 2px 0 4px;
    /* top 2px, bottom 4px */
    font-size: 13px;
    color: #555;
}

/* Đồng bộ line-height h5 và description */
.best-seller-item h5,
.best-seller-item p.clamp-2 {
    line-height: 1.25;
    /* giảm chút để gọn hơn */
}

.best-seller-description {
    height: 60px;
}

.best-product-thumbnail {
    overflow: hidden;
    border-radius: 2%;
}
</style>
