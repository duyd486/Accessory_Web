<template>
    <div id="header">
        <div class="header-top">
            Nhanh tay! Giảm giá đến 60% – Số lượng có hạn! <a href="#"> Mua ngay</a>
        </div>

        <div class="main-header">
            <div class="container-custom d-flex align-items-center justify-content-between">
                <div v-on:click="goToHome()" class="navbar-brand text-decoration-none text-dark"
                    style="cursor: pointer">
                    Victory Store <small>technology</small>
                </div>

                <div class="search-wrap">
                    <div class="search-box">
                        <input type="text" placeholder="Tìm kiếm..." v-model="searchKey" />
                    </div>
                    <button class="btn-search">
                        <i class="bi bi-search"></i>
                    </button>
                </div>

                <div class="header-actions">
                    <div class="account-dropdown">
                        <input type="checkbox" id="toggle-account" />
                        <label for="toggle-account"
                            class="account-toggle d-flex align-items-center text-decoration-none">
                            <i class="bi bi-person"></i> Tài khoản
                        </label>

                        <div class="dropdown-menu-account">
                            <a href="#" @click.prevent="goToProfile">Thông tin cá nhân</a>
                            <a href="#" @click.prevent="goToOrders">Đơn hàng</a>
                            <a v-if="token !== null" href="#" @click="logout()">
                                Đăng xuất
                                <svg width="20" height="20" aria-hidden="true" role="img" focusable="false"
                                    viewBox="0 0 32 32">
                                    <path d="M16 25.6h-9.6v-19.2h9.6v3.2h3.2v-6.4h-16v25.6h16v-6.4h-3.2z"></path>
                                    <path d="M28.8 16l-6.4-5.6v4h-11.2v3.2h11.2v4z"></path>
                                </svg>
                            </a>
                            <a v-else href="#" @click="redirectLogin()">
                                Đăng nhập
                                <svg width="20" height="20" aria-hidden="true" role="img" focusable="false"
                                    viewBox="0 0 32 32">
                                    <path d="M16 25.6h-9.6v-19.2h9.6v3.2h3.2v-6.4h-16v25.6h16v-6.4h-3.2z"></path>
                                    <path d="M28.8 16l-6.4-5.6v4h-11.2v3.2h11.2v4z"></path>
                                </svg>
                            </a>
                        </div>
                    </div>

                    <div v-on:click="goToCart()" class="cart-badge">
                        <a href="#" class="d-flex align-items-center text-decoration-none">
                            <i class="bi bi-cart"></i> Giỏ hàng
                        </a>
                        <span class="badge">{{ cartCount }}</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- Shop-category -->
        <div class="menu-row">
            <div class="container-custom">
                <a href="#" class="shop-category">
                    <i class="bi bi-grid-3x3-gap-fill"></i>
                    <div v-on:click="goToAboutUs()" class="title">Giới thiệu</div>
                </a>

                <nav class="menu-nav">
                    <ul class="nav">
                        <li v-for="category in categoriesStore.listCategory" :key="category.id"
                            class="nav-item has-dropdown">
                            <a v-if="category.parent_id === 0" class="nav-link"
                                @click.prevent="$router.push('/product?category=' + category.id)">
                                {{ category.title }}
                            </a>

                            <ul class="dropdown-menu">
                                <li v-for="child in category.children" :key="child.id">
                                    <a @click.prevent="$router.push('/product?category=' + child.id)">{{ child.title
                                    }}</a>
                                </li>
                            </ul>
                        </li>
                    </ul>
                </nav>
            </div>
        </div>
    </div>
</template>

<script setup>
import { apiHelper } from '@/helpers/axios'
import { useCategoriesStore } from '@/stores/categories'
import { mapStores } from 'pinia'
import { ref, watch } from 'vue'
import { useProductStore } from '@/stores/products'

const searchKey = ref('')
const productStore = useProductStore()

let debounceTimeout = null
watch(searchKey, (newVal) => {
    clearTimeout(debounceTimeout)
    debounceTimeout = setTimeout(() => {
        productStore.setSearchKey(newVal || null)
        productStore.selectedCategory = null
        window.history.replaceState(null, '', `/product?search_key=${encodeURIComponent(newVal)}`)
    }, 300)
})
</script>

<script>
export default {
    data() {
        return {
            token: sessionStorage.getItem('token'),
            cartCount: JSON.parse(localStorage.getItem('cart') || '[]').length,
        }
    },
    created() { },
    mounted() { },
    watch: {},
    computed: {
        ...mapStores(useCategoriesStore),
    },
    methods: {
        logout() {
            try {
                apiHelper
                    .get('/logout', {
                        headers: {
                            Authorization: 'Bearer ' + this.token,
                        },
                    })
                    .then((res) => {
                        if (res.status == 200) {
                            sessionStorage.removeItem('token')
                            this.$router.push('/login')
                        }
                    })
            } catch (error) {
                console.log(error)
            }
        },

        redirectLogin() {
            this.$router.push('login')
        },
        goToHome() {
            this.$router.push('/')
        },
        goToCart() {
            this.$router.push('/cart');
        },
        goToAboutUs() {
            this.$router.push('about-us');
        },
        goToProfile() {
            this.$router.push({
                path: '/profile',
                query: { tab: 'profile' }
            })
        },

        goToOrders() {
            this.$router.push({
                path: '/profile',
                query: { tab: 'orders' }
            })
        },

    },
}
</script>

<style scoped>
:root {
    --primary: #5b3df5;
    --primary-light: #ede9ff;
    --primary-soft: #f5f2ff;
    --text-dark: #151515;
}

body {
    font-family: 'Outfit', sans-serif;
    background: #fff;
}

.header-top {
    background:
        linear-gradient(
            135deg,
            #5b3df5,
            #7c3aed
        );
    text-align: center;
    padding: 14px 0;
    font-size: 13px;
    font-weight: 600;
    color: white;
    letter-spacing: 0.3px;
}

.header-top a {
    color: white;
    font-weight: 700;
    margin-left: 4px;
}

.main-header {
    background:
        rgba(255,255,255,0.75);
    backdrop-filter: blur(14px);
    padding: 24px 0;
    border-bottom:
        1px solid rgba(230,230,230,0.7);
    position: relative;
    z-index: 10000;
}

.container-custom {
    max-width: 1250px;
    margin: 0 auto;
    padding: 0 10px;
}

.navbar-brand {
    font-weight: 800;
    font-size: 30px;
    display: flex;
    align-items: center;
    gap: 10px;
    color: #151515;
    transition: all 0.25s ease;
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
    height: 50px;
    background:
        rgba(255,255,255,0.9);
    border-radius: 14px;
    border:
        1px solid rgba(230,230,230,0.8);
    backdrop-filter: blur(10px);
    box-shadow:
        0 10px 30px rgba(91, 61, 245, 0.08);
    transition: all 0.25s ease;
    overflow: hidden;
}

.search-box input {
    height: 100%;
    border: none;
    outline: none;
    flex: 1;
    padding: 0 20px;
    font-size: 15px;
    background: transparent;
    box-shadow: none;
    color: #151515;
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
    width: 45px;
    height: 45px;
    border-radius: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
    background:
        linear-gradient(
            135deg,
            #5b3df5,
            #7c3aed
        );
    color: #fff;
    border: none;
    margin-right: 4px;
    transition: all 0.25s ease;
    box-shadow:
        0 8px 20px rgba(91, 61, 245, 0.22);
}

.btn-search:hover {
    opacity: 0.9;
}

/* shop-category */
.menu-row {
    background:
        rgba(255,255,255,0.82);
    backdrop-filter: blur(12px);
    border-bottom:
        1px solid rgba(230,230,230,0.7);
    position: relative;
    z-index: 1000;
}

.menu-row .container-custom {
    display: flex;
    align-items: center;
    gap: 10px;
    position: relative;
  
}

/* Shop Category (Giới thiệu) */
.shop-category {
    height: 50px;
    min-width: 130px;
    width:auto;
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 0 18px;
    border-radius: 18px;
    background:
        linear-gradient(
            135deg,
            #5b3df5,
            #7c3aed
        );
    color: white;
    text-decoration: none;
    position: relative;
    
    box-shadow:
        0 12px 28px rgba(91, 61, 245, 0.22);
    transition: all 0.3s ease;
}

/* Giữ hover nhẹ, không gạch chân, không đổi màu */
.shop-category:hover {
    transform: translateY(-3px);
    box-shadow:
        0 18px 35px rgba(91, 61, 245, 0.3);
    color: white;
}

/* Bỏ hoàn toàn hiệu ứng gạch chân */
.shop-category::after {
    display: none;
}

/* MENU CHÍNH - NAVIGATION */
.menu-nav {
    position: relative;
}

.menu-nav .nav {
    display: flex;
    align-items: center;
    gap: 10px;
    list-style: none;
    
    padding: 0;
    flex-wrap: nowrap !important;
    width: max-content;
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
    margin-left: 15px;
    white-space: nowrap;

}

/* Giữ hiệu ứng gạch chân động cho menu cha */
.menu-nav .nav-link::after {
    content: '';
    position: absolute;
    left: 0;
    bottom: -2px;
    height: 2px;
    width: 0;
    border-radius: 999px;
    background:
        linear-gradient(
            90deg,
            #5b3df5,
            #7c3aed
        );
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
    min-width: 230px;
    background:
        rgba(255,255,255,0.92);
    backdrop-filter: blur(14px);
    border:
        1px solid rgba(230,230,230,0.8);
    box-shadow:
        0 18px 40px rgba(91, 61, 245, 0.12);
    border-radius: 20px;
    padding: 12px 0;
    z-index: 10000;
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
    z-index: 10000;
}

.header-actions a,
.header-actions label {
    color: #151515;
    font-weight: 600;
    cursor: pointer;
    text-decoration: none;
    transition: all 0.25s ease;
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
    top: -8px;
    right: -10px;
    background:
        linear-gradient(
            135deg,
            #5b3df5,
            #7c3aed
        );
    color: white;
    font-size: 11px;
    font-weight: 700;
    padding: 5px 8px;
    border-radius: 999px;
    box-shadow:
        0 4px 12px rgba(91, 61, 245, 0.3);
}

.account-dropdown {
    position: relative;
    z-index: 10000;
}

#toggle-account {
    display: none;
}

.dropdown-menu-account {
    display: none;
    position: absolute;
    top: calc(100% + 10px);
    left: 0;
    background: #fff;
    border: 1px solid #eaeaea;
    border-radius: 6px;
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
    padding: 10px 0;
    min-width: 180px;
    z-index: 10000;
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

.container {
    max-width: 1200px;
    margin: auto;
    padding: 20px 20px 0 20px;
}
</style>