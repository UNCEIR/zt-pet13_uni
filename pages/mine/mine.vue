<template>
    <view class="page">
        <!-- 顶部信息卡片：绿渐变，头像 + 名宠 + 设备 ID + 切换设备 -->
        <view class="user-card">
            <image class="user-avatar" src="/static/images/pet-avatar.png" mode="aspectFill" />
            <view class="user-info">
                <text class="pet-name">{{ petName }}</text>
                <view class="device-row">
                    <view class="device-icon-css">
                        <view class="css-phone-icon"></view>
                    </view>
                    <text class="device-imei">{{ deviceImei }}</text>
                </view>
            </view>
            <view class="switch-wrap" @tap="goSwitchDevice">
                <image class="icon-switch" src="/static/images/icons/切换设备.png" mode="aspectFit" />
                <text class="switch-text">切换设备</text>
            </view>
        </view>

        <!-- 爱宠身份卡：独立卡片，与下方保留间隔 -->
        <view class="menu-card">
            <view class="menu-item" @tap="onIdentityCard">
                <image class="menu-icon" src="/static/images/icons/爱宠身份卡.png" mode="aspectFit" />
                <text class="menu-text">爱宠身份卡</text>
                <view class="arrow"></view>
            </view>
        </view>
        <!-- 通讯录 / 亲情号 / SOS号码 -->
        <view class="menu-card">
            <view class="menu-item" @tap="onAddressBook">
                <image class="menu-icon" src="/static/images/icons/通讯录.png" mode="aspectFit" />
                <text class="menu-text">通讯录</text>
                <view class="arrow"></view>
            </view>
            <view class="menu-item" @tap="onFamilyNumber">
                <image class="menu-icon" src="/static/images/icons/亲情号.png" mode="aspectFit" />
                <text class="menu-text">亲情号</text>
                <view class="arrow"></view>
            </view>
            <view class="menu-item" @tap="onSOS">
                <image class="menu-icon" src="/static/images/icons/SOS号码.png" mode="aspectFit" />
                <text class="menu-text">SOS号码</text>
                <view class="arrow"></view>
            </view>
        </view>

        <!-- 第二组功能列表 -->
        <view class="menu-card">
            <view class="menu-item" @tap="onManual">
                <image class="menu-icon" src="/static/images/icons/电子说明书.png" mode="aspectFit" />
                <text class="menu-text">电子说明书</text>
                <view class="arrow"></view>
            </view>
            <view class="menu-item" @tap="onChangePwd">
                <image class="menu-icon" src="/static/images/icons/修改密码.png" mode="aspectFit" />
                <text class="menu-text">修改密码</text>
                <view class="arrow"></view>
            </view>
            <view class="menu-item" @tap="onSwitchLang">
                <image class="menu-icon" src="/static/images/icons/切换语言.png" mode="aspectFit" />
                <text class="menu-text">切换语言</text>
                <view class="arrow"></view>
            </view>
            <view class="menu-item" @tap="onVersion">
                <image class="menu-icon" src="/static/images/icons/软件版本.png" mode="aspectFit" />
                <text class="menu-text">软件版本</text>
                <view class="arrow"></view>
            </view>
            <view class="menu-item menu-item-logout" @tap="onLogout">
                <image class="menu-icon" src="/static/images/icons/退出账号.png" mode="aspectFit" />
                <text class="menu-text">退出账号</text>
                <view class="arrow"></view>
            </view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            petName: '名宠',
            deviceImei: '862506074971204'
        };
    },
    onLoad() {
        this.refreshDeviceImei();
    },
    onShow() {
        this.refreshDeviceImei();
    },
    methods: {
        refreshDeviceImei() {
            const device = app.globalData._selectedDevice;
            const imei = device ? device.imei : app.globalData.defaultDeviceImei || '862506074971204';
            this.setData({
                deviceImei: imei
            });
        },

        goSwitchDevice() {
            uni.navigateTo({
                url: '/pages/device-select/device-select'
            });
        },

        onIdentityCard() {
            uni.navigateTo({
                url: '/pages/pet-idcard/pet-idcard'
            });
        },

        onAddressBook() {
            uni.navigateTo({
                url: '/pages/contacts/contacts'
            });
        },

        onFamilyNumber() {
            uni.navigateTo({
                url: '/pages/family-number/family-number'
            });
        },

        onSOS() {
            uni.showToast({
                title: '敬请期待',
                icon: 'none'
            });
        },

        onManual() {
            uni.navigateTo({
                url: '/pages/manual/manual'
            });
        },

        onChangePwd() {
            uni.showToast({
                title: '敬请期待',
                icon: 'none'
            });
        },

        onSwitchLang() {
            uni.navigateTo({
                url: '/pages/language/language'
            });
        },

        onVersion() {
            uni.navigateTo({
                url: '/pages/version/version'
            });
        },

        onLogout() {
            uni.showToast({
                title: '敬请期待',
                icon: 'none'
            });
        }
    }
};
</script>
<style>
@import './mine.css';
</style>
