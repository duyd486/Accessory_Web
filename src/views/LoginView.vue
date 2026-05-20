<template>
    <div class="signin-container">
        <div class="signin-card">

            <div class="signin-banner">
                <div class="overlay"></div>

                <div class="banner-content">

                    <div class="banner-text">
                        <p>Accessory Customer Website</p>
                        <h2>
                            Khám phá không gian mua sắm dành riêng cho bạn.
                        </h2>
                    </div>
                </div>
            </div>

            <!-- RIGHT PANEL -->
            <div class="signin-form-wrapper">

                <h2 class="title">Đăng nhập</h2>

                <p class="subtitle">
                    Chào mừng bạn quay trở lại! 
                    Mọi nhu cầu mua sắm của bạn đều sẽ được gói gọn trong một nơi.
                </p>
                
                <form class="form">
                    <div class="input-group">
                        <label>Nhập Email của bạn</label>
                        <input
                            type="email"
                            v-model="email"
                            required
                        />
                    </div>

                    <div class="input-group">
                        <label>Nhập mật khẩu</label>
                        <input
                            type="password"
                            v-model="password"
                            required
                        />
                    </div>

                    <button
                        v-on:click="login()"
                        type="submit"
                        class="btn-primary"
                        :disabled="loading"
                    >
                        <span v-if="loading">Đang đăng nhập...</span>
                        <span v-else>Đăng nhập</span>
                    </button>
                </form>

                <div class="divider">
                    <span>hoặc tiếp tục với</span>
                </div>

                <div class="google-login">
                    <GoogleLoginComponent :title="'Đăng nhập'" />
                </div>

                <p class="footer-text">
                    Bạn chưa có tài khoản?
                    <button @click="navigateToSignup">Đăng kí ngay</button>
                </p>

            </div>
        </div>
    </div>
</template>

<script setup>
import GoogleLoginComponent from '@/components/GoogleLoginComponent.vue'
import { apiHelper } from '@/helpers/axios';
import HomeView from './HomeView.vue';
</script>

<script>
export default {
    data() {
        return {
            loading: false,
            email: '',
            password: '',
        }
    },
    methods: {
        navigateToSignup() {
            this.$router.push('/signup')
        },

        login() {
            try {
                this.loading = true;
                const formData = new FormData();
                formData.append('email', this.email);
                formData.append('password', this.password);

                apiHelper.post('/login', formData).then((res) => {
                    if (res.status == 200) {
                        sessionStorage.setItem('token', res.data.data.token);
                        this.$router.push('/home');
                    }
                }).catch((error) => {
                    console.log(error);
                    this.loading = false;
                });

            } catch (error) {
                console.log(error);
            }
        }
    },
}
</script>

<style scoped>
.signin-container {
    width: 100%;
    min-height: 100vh;
    background: #f3efff;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 30px;
    font-family: 'Segoe UI', sans-serif;
}

.signin-card {
    width: 1100px;
    min-height: 680px;
    background: white;
    border-radius: 24px;
    overflow: hidden;
    display: flex;
    box-shadow:
        0 15px 40px rgba(116, 84, 255, 0.15);
}

/* LEFT SIDE */

.signin-banner {
    width: 45%;
    position: relative;
    overflow: hidden;
    background:
        radial-gradient(circle at top left,
            rgba(255,255,255,0.9),
            rgba(255,255,255,0) 25%),

        radial-gradient(circle at center,
            rgba(79, 70, 229, 0.95),
            rgba(124, 58, 237, 0.9),
            rgba(221, 214, 254, 0.8));

    display: flex;
    align-items: flex-end;
    padding: 40px;
}

.overlay {
    position: absolute;
    inset: 0;
    backdrop-filter: blur(10px);
}

.banner-content {
    position: relative;
    z-index: 2;
    width: 100%;
}


.banner-text p {
    color: rgba(255,255,255,0.85);
    font-size: 15px;
    margin-bottom: 10px;
}

.banner-text h2 {
    color: white;
    font-size: 36px;
    line-height: 1.25;
    font-weight: 700;
    max-width: 360px;
}


.signin-form-wrapper {
    width: 55%;
    padding: 70px 80px;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.title {
    font-size: 38px;
    font-weight: 700;
    color: #111;
    margin-bottom: 12px;
}

.subtitle {
    color: #777;
    font-size: 15px;
    line-height: 1.6;
    margin-bottom: 40px;
    max-width: 430px;
}

.form {
    width: 100%;
}

.input-group {
    margin-bottom: 24px;
}

.input-group label {
    display: block;
    font-size: 14px;
    font-weight: 600;
    margin-bottom: 10px;
    color: #222;
}

.input-group input {
    width: 500px;
    height: 52px;
    border: 1px solid #ddd;
    padding: 16px;
    font-size: 14px;
    transition: all 0.2s ease;
    background: #fff;
    border-radius: 10px !important;
}

.input-group input:focus {
    outline: none;
    border-color: #6b4eff;
    box-shadow: 0 0 0 4px rgba(107, 78, 255, 0.1);
    border-radius: 10px !important;
}

.btn-primary {
    width: 100%;
    height: 52px;
    border: none;
    border-radius: 10px;
    background: #5b3df5;
    color: white;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    transition: 0.2s ease;
    margin-top: 10px;

    box-shadow:
        0 10px 20px rgba(91, 61, 245, 0.25);
}

.btn-primary:hover {
    background: #4c2ff0;
    transform: translateY(-1px);
}

.btn-primary:disabled {
    opacity: 0.7;
    cursor: not-allowed;
}

.divider {
    display: flex;
    align-items: center;
    gap: 12px;
    margin: 35px 0 25px;
    color: #999;
    font-size: 13px;
}

.divider::before,
.divider::after {
    content: '';
    flex: 1;
    height: 1px;
    background: #e4e4e4;
}

.google-login {
    width: 100%;
}

.footer-text {
    margin-top: 28px;
    text-align: center;
    color: #888;
    font-size: 14px;
}

.footer-text button {
    border: none;
    background: transparent;
    color: #5b3df5;
    font-weight: 600;
    cursor: pointer;
}

.footer-text button:hover {
    text-decoration: underline;
}

/* RESPONSIVE */

@media (max-width: 900px) {
    .signin-card {
        flex-direction: column;
        width: 100%;
    }

    .signin-banner {
        width: 100%;
        min-height: 300px;
    }

    .signin-form-wrapper {
        width: 100%;
        padding: 50px 35px;
    }

    .banner-text h2 {
        font-size: 28px;
    }

    .title {
        font-size: 30px;
    }
}
</style>