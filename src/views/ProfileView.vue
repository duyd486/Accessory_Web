<template>
    <HeaderComponent />

    <div class="body container py-5">
        <h1 class="my-account">THÔNG TIN TÀI KHOẢN</h1>

        <div class="row mt-4">
            <!-- LEFT NAV TABS -->
            <div class="col-md-3">
                <div class="sidebar">
                    <div class="user-info d-flex align-items-center px-3 py-3">
                        <img :src="user.avatar || defaultAvatar" class="avatar-img" alt="Avatar" />
                        <div class="ms-3">
                            <div class="hello-text">Hello!</div>
                            <p class="username">{{ user.name }}</p>
                        </div>
                    </div>

                    <ul class="menu-list">

                        <!-- TAB PROFILE -->
                        <li :class="{ active: activeMainTab === 'profile' }" @click="setMainTab('profile')">
                            Thông tin cá nhân
                        </li>

                        <!-- TAB ORDERS -->
                        <li :class="{ active: activeMainTab === 'orders' }" @click="setMainTab('orders')">
                            Đơn hàng
                        </li>

                        <li>
                            Đăng xuất
                        </li>
                    </ul>
                </div>
            </div>

            <!-- RIGHT CONTENT -->
            <div class="col-md-9">

                <!-- TAB: THÔNG TIN CÁ NHÂN -->
                <div v-if="activeMainTab === 'profile'" class="content-box">
                    <UserInfoComponent :user="user" />
                </div>

                <!-- TAB: ĐƠN HÀNG -->
                <div v-if="activeMainTab === 'orders'" class="content-box">
                    <OrderHistoryComponent />
                </div>

            </div>
        </div>
    </div>

    <FooterComponent />
</template>

<script setup>
import HeaderComponent from '../components/HeaderComponent.vue'
import FooterComponent from '@/components/FooterComponent.vue'
import OrderHistoryComponent from '@/components/OrderHistoryComponent.vue'
import UserInfoComponent from '@/components/UserInfoComponent.vue'

import { ref, onMounted, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()
const defaultAvatar = '../assets/MyLogo.jpg' // Nếu user chưa có avatar
const user = ref({})

// Tab chính
const activeMainTab = ref("profile")

// Hàm đổi tab + cập nhật URL
const setMainTab = (tab) => {
    activeMainTab.value = tab
    router.replace({ query: { tab } })  // replace để không spam history
}

// Khi load trang → đọc URL để set tab mặc định
onMounted(() => {
    if (route.query.tab === 'orders' || route.query.tab === 'profile') {
        activeMainTab.value = route.query.tab
    }

    // Lấy thông tin user
    const storedUser = localStorage.getItem('auth') || sessionStorage.getItem('auth')
    if (storedUser) {
        user.value = JSON.parse(storedUser)
    }
})

// WATCH: Khi URL đổi → tab đổi theo (QUAN TRỌNG)
watch(
    () => route.query.tab,
    (newTab) => {
        if (newTab && ['profile', 'orders'].includes(newTab)) {
            activeMainTab.value = newTab
        }
    }
)
</script>
<script>
export default {
    props: {
        product: Object,
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
    }
}
</script>


<style scoped>
body {
    font-family: Arial, sans-serif;
    background: #fafafe;
}

.container {
    max-width: 1150px;
    margin: auto;
    padding: 0 20px;
}

.my-account {
    font-size: 32px;
    font-weight: 900;
    text-align: center;
    margin-bottom: 45px;
    color: #151515;
}

/* Sidebar */
.sidebar {
    background:
        rgba(255,255,255,0.82);
    backdrop-filter: blur(14px);

    border:
        1px solid rgba(230,230,230,0.8);

    border-radius: 24px;

    overflow: hidden;

    box-shadow:
        0 12px 35px rgba(91, 61, 245, 0.08);
}

/* User Info */
.user-info {
    padding: 24px !important;

    border-bottom:
        1px solid rgba(235,235,235,0.9);

    background:
        linear-gradient(
            135deg,
            rgba(91,61,245,0.06),
            rgba(124,58,237,0.03)
        );
}

/* Avatar */
.avatar-img {
    width: 68px;
    height: 68px;
    border-radius: 50%;
    object-fit: cover;

    border: 3px solid #fff;

    box-shadow:
        0 8px 20px rgba(91, 61, 245, 0.18);
}

/* Hello text */
.hello-text {
    font-size: 14px;
    font-weight: 500;
    color: #777;
    margin-bottom: 4px;
}

/* Username */
.username {
    font-size: 18px;
    font-weight: 700;
    color: #111;
    margin: 0;
}

/* Menu */
.menu-list {
    list-style: none;
    margin: 0;
    padding: 12px;
}

.menu-list li {
    position: relative;

    display: flex;
    align-items: center;

    padding: 14px 18px;

    margin-bottom: 8px;

    border-radius: 16px;

    cursor: pointer;

    font-size: 15px;
    font-weight: 500;

    color: #666;

    transition: all .25s ease;
}

/* Hover */
.menu-list li:hover {
    background:
        rgba(91, 61, 245, 0.08);

    color: #5b3df5;

    transform: translateX(3px);
}

/* Active */
.menu-list li.active {
    background:
        linear-gradient(
            135deg,
            #5b3df5,
            #7c3aed
        );

    color: white;

    font-weight: 600;

    box-shadow:
        0 10px 24px rgba(91, 61, 245, 0.22);
}

/* Remove old effect */
.menu-list li::before {
    display: none;
}
</style>
