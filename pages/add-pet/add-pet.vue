<template>
    <view class="page">
        <!-- 扫一扫设备号（浅绿底色+深绿文字） -->
        <view class="scan-wrap">
            <view class="scan-btn" @tap="onScan">
                <text class="scan-text">扫一扫设备号</text>
                <image class="scan-icon" src="/static/images/icons/热点围栏.png" mode="aspectFit" />
            </view>
        </view>

        <!-- 宠物头像（与首页一致：圆形+右下角小图标） -->
        <view class="avatar-section">
            <view class="avatar-wrap">
                <image class="avatar" :src="avatarUrl || '/static/images/pet-avatar.png'" mode="aspectFill" />
                <view class="edit-icon" @tap="onChangeAvatar">
                    <text class="edit-icon-text">✎</text>
                </view>
            </view>
        </view>

        <!-- 设备信息（头像与宠物名称之间，始终展示） -->
        <view class="device-info">
            <view class="device-row">
                <text class="device-label">IMEI号</text>
                <text class="device-value">{{ deviceImei || '—' }}</text>
            </view>
            <view class="device-row">
                <text class="device-label">设备型号</text>
                <text class="device-value">{{ deviceModel || '—' }}</text>
            </view>
        </view>

        <!-- 宠物信息表单（每项独立卡片） -->
        <view class="form-card">
            <view class="form-item">
                <text class="label">宠物名称</text>
                <input class="input" placeholder="请输入宠物名称" :value="name" @input="onNameInput" />
            </view>
            <view class="form-item" @tap="goGenderSelect">
                <text class="label">性别</text>
                <text class="value">{{ gender || '请选择' }}</text>
                <view class="arrow"></view>
            </view>
            <picker mode="date" :value="dateValue" fields="month" start="2010-01" end="2030-12" @change="onDateChange">
                <view class="form-item">
                    <text class="label">出身年月</text>
                    <text class="value">{{ birthText || '请选择' }}</text>
                    <view class="arrow"></view>
                </view>
            </picker>
            <view class="form-item" @tap="goBreedSelect">
                <text class="label">品种</text>
                <text class="value">{{ breed || '请选择' }}</text>
                <view class="arrow"></view>
            </view>
        </view>

        <view class="btn-wrap">
            <view :class="'btn-primary ' + (!canSubmit ? 'disabled' : '')" @tap="onSubmit">确定</view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            avatarUrl: '',
            deviceImei: '862506074976309',
            deviceModel: 'PT13',
            name: '喵喵',
            gender: '公',
            birthYear: 2025,
            birthMonth: 6,
            dateValue: '2025-06',
            birthText: '2025 年 6 月',
            breed: '美短猫',
            canSubmit: false
        };
    },
    onLoad() {},
    onShow() {
        const g = app.globalData;
        const device = g._selectedDevice;
        if (device) {
            this.setData({
                deviceImei: device.imei,
                deviceModel: device.model || 'PT13'
            });
            g._selectedDevice = null;
        }
        const breed = g._selectedBreed;
        if (breed) {
            this.setData({
                breed
            });
            g._selectedBreed = null;
        }
        const gender = g._selectedGender;
        if (gender) {
            this.setData({
                gender
            });
            g._selectedGender = null;
        }
        this.checkSubmit();
    },
    methods: {
        onScan() {
            uni.scanCode({
                success: (res) => {
                    const imei = (res.result || '').replace(/\D/g, '').slice(0, 15);
                    if (imei) {
                        this.setData({
                            deviceImei: imei,
                            deviceModel: 'PT13'
                        });
                        this.checkSubmit();
                    } else {
                        uni.showToast({
                            title: '未识别到设备号',
                            icon: 'none'
                        });
                    }
                }
            });
        },

        onChangeAvatar() {
            uni.chooseMedia({
                count: 1,
                mediaType: ['image'],
                sourceType: ['album', 'camera'],
                success: (res) => {
                    this.setData({
                        avatarUrl: res.tempFiles[0].tempFilePath
                    });
                }
            });
        },

        onNameInput(e) {
            this.setData({
                name: e.detail.value
            });
            this.checkSubmit();
        },

        checkSubmit() {
            const { deviceImei, name, gender, birthText, breed } = this;
            const canSubmit = !!(deviceImei && name.trim() && gender && birthText && breed);
            this.setData({
                canSubmit
            });
        },

        goGenderSelect() {
            uni.navigateTo({
                url: '/pages/gender-select/gender-select'
            });
        },

        onDateChange(e) {
            const v = e.detail.value;
            if (!v) {
                return;
            }
            const [y, m] = v.split('-').map(Number);
            this.setData({
                birthYear: y,
                birthMonth: m,
                dateValue: v,
                birthText: `${y} 年 ${m} 月`
            });
            this.checkSubmit();
        },

        goBreedSelect() {
            uni.navigateTo({
                url: '/pages/breed-select/breed-select'
            });
        },

        onSubmit() {
            if (!this.canSubmit) {
                return;
            }
            const pet = {
                id: Date.now(),
                name: this.name,
                gender: this.gender,
                birthYear: this.birthYear,
                birthMonth: this.birthMonth,
                breed: this.breed,
                imei: this.deviceImei,
                model: this.deviceModel,
                avatar: this.avatarUrl
            };
            app.globalData.pets = app.globalData.pets || [];
            app.globalData.pets.push(pet);
            uni.setStorageSync('pets', app.globalData.pets);
            uni.showToast({
                title: '添加成功',
                icon: 'success'
            });
            setTimeout(() => {
                uni.switchTab({
                    url: '/pages/index/index'
                });
            }, 800);
        }
    }
};
</script>
<style>
@import './add-pet.css';
</style>
