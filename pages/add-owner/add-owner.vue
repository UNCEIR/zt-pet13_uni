<template>
    <view class="page">
        <view class="form">
            <view class="form-item">
                <text class="label">创建者名称</text>
                <input class="input" placeholder="请输入成员名称" :value="name" @input="onNameInput" />
            </view>
            <view class="form-item">
                <text class="label">手机号</text>
                <input class="input" type="number" maxlength="11" placeholder="请输入手机号码" :value="phone" @input="onPhoneInput" />
            </view>
            <view class="form-item">
                <text class="label">验证码</text>
                <view class="input-with-btn">
                    <input class="input" type="number" maxlength="6" placeholder="请输入验证码" :value="code" @input="onCodeInput" />
                    <text :class="'code-btn ' + (!canSendCode ? 'disabled' : '')" @tap="sendCode">{{ codeText }}</text>
                </view>
            </view>
            <view class="form-item select-item">
                <text class="label">选择设备</text>
                <view class="select-box" @tap="goDeviceSelect">
                    <text class="select-value" v-if="selectedImei">{{ selectedImei }}</text>
                    <text class="select-placeholder" v-else>请选择授权设备</text>
                    <view class="arrow-wrap">
                        <view class="arrow"></view>
                    </view>
                </view>
                <view class="device-dropdown" v-if="deviceListVisible">
                    <scroll-view class="device-list-scroll" scroll-y>
                        <view class="device-list-item" @tap="onSelectDevice" :data-index="index" v-for="(item, index) in devices" :key="index">
                            <text class="device-imei">{{ item.imei }}</text>

                            <text class="device-model">{{ item.model }}</text>

                            <view class="arrow"></view>
                        </view>
                    </scroll-view>
                </view>
            </view>
        </view>

        <view class="btn-wrap">
            <view class="btn-primary" @tap="onSubmit">立即登录</view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            name: '',
            phone: '',
            code: '',
            selectedImei: '',
            selectedDevice: null,
            deviceListVisible: false,
            devices: [],
            canSendCode: false,
            canSubmit: false,
            codeText: '发送验证码',
            countdown: 0
        };
    },
    onLoad() {
        this.setData({
            devices: getApp().globalData.devices || []
        });
    },
    onShow() {
        const device = getApp().globalData._selectedDevice;
        if (device) {
            getApp().globalData._selectedDevice = null;
            this.setData(
                {
                    selectedImei: device.imei,
                    selectedDevice: device
                },
                () => this.checkSubmit()
            );
        }
    },
    methods: {
        onNameInput(e) {
            this.setData(
                {
                    name: e.detail.value
                },
                () => this.checkSubmit()
            );
        },

        onPhoneInput(e) {
            const phone = e.detail.value;
            const canSend = /^1\d{10}$/.test(phone);
            this.setData(
                {
                    phone,
                    canSendCode: canSend
                },
                () => this.checkSubmit()
            );
        },

        onCodeInput(e) {
            this.setData(
                {
                    code: e.detail.value
                },
                () => this.checkSubmit()
            );
        },

        checkSubmit() {
            const { name, phone, code, selectedImei } = this;
            const canSubmit = !!(name && String(name).trim() && /^1\d{10}$/.test(phone) && /^\d{6}$/.test(code) && selectedImei);
            this.setData({
                canSubmit
            });
        },

        sendCode() {
            if (this.countdown > 0) {
                return;
            }
            this.setData({
                countdown: 60,
                codeText: '60s'
            });
            const t = setInterval(() => {
                let n = this.countdown - 1;
                if (n <= 0) {
                    clearInterval(t);
                    this.setData({
                        countdown: 0,
                        codeText: '发送验证码',
                        canSendCode: /^1\d{10}$/.test(this.phone)
                    });
                } else {
                    this.setData({
                        countdown: n,
                        codeText: n + 's',
                        canSendCode: false
                    });
                }
            }, 1000);
            uni.showToast({
                title: '验证码已发送',
                icon: 'none'
            });
        },

        goDeviceSelect() {
            this.setData({
                deviceListVisible: !this.deviceListVisible
            });
        },

        onSelectDevice(e) {
            const index = e.currentTarget.dataset.index;
            const devices = this.devices;
            const device = devices != null && devices[index] != null ? devices[index] : null;
            if (!device) {
                return;
            }
            this.setData(
                {
                    selectedImei: device.imei,
                    selectedDevice: device,
                    deviceListVisible: false
                },
                () => this.checkSubmit()
            );
        },

        onSubmit() {
            const ownerName = (this.name || '').trim() || '主人';
            app.globalData.hasOwner = true;
            app.globalData.ownerName = ownerName;
            uni.setStorageSync('ownerName', ownerName);
            uni.showToast({
                title: '登录成功',
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
@import './add-owner.css';
</style>
