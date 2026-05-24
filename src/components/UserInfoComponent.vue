<template>
    <div class="content-box">

        <h5 class="mb-3">Thông tin cá nhân</h5>

        <form>
            <div class="mb-3">
                <label>Họ tên</label>
                <input type="text" class="form-control" v-model="formData.name" />
            </div>

            <div class="mb-3">
                <label>Địa chỉ Email</label>
                <input type="text" class="form-control" v-model="formData.email" />
            </div>

            <div class="mb-4">
                <label>Số điện thoại</label>
                <input type="text" class="form-control" v-model="formData.phone" />
            </div>

            <div class="mb-4">
                <label>Địa chỉ</label>
                <input type="text" class="form-control" v-model="formData.address" />
            </div>

            <h5 class="mb-3">Thay đổi mật khẩu</h5>

            <div class="mb-3">
                <label>Mật khẩu hiện tại</label>
                <input type="password" class="form-control" v-model="formData.current_password" />
            </div>

            <div class="mb-3">
                <label>Mật khẩu mới</label>
                <input type="password" class="form-control" v-model="formData.new_password" />
            </div>

            <div class="mb-4">
                <label>Xác nhận mật khẩu mới</label>
                <input type="password" class="form-control" v-model="formData.confirm_password" />
            </div>

            <button type="button" class="btn-save w-100" @click="saveChanges">Lưu thay đổi</button>
        </form>

    </div>
</template>

<script setup>
import { apiHelper } from '@/helpers/axios';
import { ref, watch } from 'vue'

const props = defineProps({
    user: Object
});

// formData
const formData = ref({
    name: props.user?.name || '',
    email: props.user?.email || '',
    phone: props.user?.phone || '',
    address: props.user?.address || '',
    current_password: '',
    new_password: '',
    confirm_password: ''
});

// khi prop user thay đổi → update form
watch(() => props.user, (newUser) => {
    if (newUser) {
        formData.value.name = newUser.name || ''
        formData.value.email = newUser.email || ''
        formData.value.phone = newUser.phone || ''
        formData.value.address = newUser.address || ''
    }
}, { immediate: true })

// function saveChanges
const saveChanges = async () => {
    try {
        const payload = {
            name: formData.value.name,
            email: formData.value.email,
            phone: formData.value.phone,
            address: formData.value.address
        };

        // chỉ thêm mật khẩu nếu người dùng nhập
        if (formData.value.current_password || formData.value.new_password || formData.value.confirm_password) {
            payload.current_password = formData.value.current_password;
            payload.new_password = formData.value.new_password;
            payload.new_password_confirmation = formData.value.confirm_password;
        }

        const res = await apiHelper.post('/update-profile', payload, {
            headers: {
                Authorization: 'Bearer ' + sessionStorage.getItem('token')
            }
        });

        if (res.status === 200) {
            alert(res.data.message);
            emit('updated-user', res.data.user);
        }
    } catch (error) {
        console.error(error);
        alert(error.response?.data?.message || 'Có lỗi xảy ra');
    }
}

// emit
const emit = defineEmits(['updated-user']);
</script>

<style scoped>
.content-box {
    border: 1px solid #eee;
    border-radius: 20px;
    padding: 32px;
    background: #fff;
    box-shadow: 0 10px 30px rgba(0,0,0,0.04);
}

.content-box h5 {
    font-size: 22px;
    font-weight: 700;
    margin-bottom: 24px;
    color: #151515;
}

label {
    font-size: 14px;
    font-weight: 600;
    margin-bottom: 8px;
    display: block;
    color: #333;
}

.form-control {
    height: 52px;
    border-radius: 14px;
    border: 1px solid #e5e7eb;
    padding: 0 18px;
    font-size: 15px;
    transition: all 0.25s ease;
    box-shadow: none;
}

.form-control:focus {
    border-color: #5b3df5;
    box-shadow:
        0 0 0 4px rgba(91, 61, 245, 0.12);
}

.btn-save {
    height: 52px;
    border-radius: 14px;
    background: linear-gradient(
        135deg,
        #5b3df5,
        #7c3aed
    );
    color: #fff;
    font-weight: 600;
    border: none;
    transition: all 0.25s ease;
    box-shadow:
        0 10px 24px rgba(91,61,245,.22);
}

.btn-save:hover {
    transform: translateY(-2px);
    opacity: .95;
}
</style>
