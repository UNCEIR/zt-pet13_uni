<template>
    <view class="page">
        <!-- 顶部装饰爪印 -->
        <view class="paw-decoration top">
            <text class="paw">🐾</text>
            <text class="paw">🐾</text>
            <text class="paw">🐾</text>
        </view>

        <!-- 宠物信息卡片 -->
        <view class="idcard-header">
            <view class="avatar-section">
                <view class="avatar-ring">
                    <image class="pet-avatar" :src="petAvatar || '/static/images/pet-avatar.png'" mode="aspectFill" />
                    <view class="avatar-badge">
                        <text class="badge-icon">⭐</text>
                    </view>
                </view>
            </view>
            <view class="pet-name-wrap">
                <text class="pet-name">{{ petName }}</text>
                <text class="pet-emoji">🐾</text>
            </view>
            <view class="pet-brief">
                <text class="breed">{{ petBreed || '萌宠' }}</text>
                <text class="dot">·</text>
                <text class="age">{{ petAge || '未知年龄' }}</text>
            </view>
            <view class="pet-tag">
                <text class="tag-icon">{{ genderIcon }}</text>
                <text class="tag-text">{{ genderText }}</text>
            </view>
        </view>

        <!-- 主人联系方式卡片 -->
        <view class="info-card owner-card">
            <view class="card-title">
                <text class="title-icon">🐾</text>
                <text class="title-text">主人联系方式</text>
                <text class="title-icon">🐾</text>
            </view>
            <view class="card-content">
                <view class="info-row">
                    <text class="info-label">👤 主人</text>
                    <text class="info-value">{{ ownerName || '宠主' }}</text>
                </view>
                <view class="info-row">
                    <text class="info-label">📱 电话</text>
                    <text class="info-value">{{ ownerPhone || '未设置' }}</text>
                </view>
            </view>
            <view class="action-btn primary" @tap="callOwner" v-if="ownerPhone">
                <text class="btn-icon">📲</text>
                <text class="btn-text">一键拨打</text>
            </view>
        </view>

        <!-- 设备追踪信息卡片 -->
        <view class="info-card device-card">
            <view class="card-title">
                <text class="title-icon">📡</text>
                <text class="title-text">设备追踪信息</text>
                <text class="title-icon">📡</text>
            </view>
            <view class="card-content">
                <view class="info-row">
                    <text class="info-label">🔍 IMEI</text>
                    <text class="info-value imei">{{ deviceImei }}</text>
                </view>
                <view class="info-row">
                    <text class="info-label">📶 状态</text>
                    <text :class="'info-value status ' + deviceStatus">{{ deviceStatusText }}</text>
                </view>
            </view>
            <view class="action-btn secondary" @tap="viewLocation">
                <text class="btn-icon">📍</text>
                <text class="btn-text">查看实时位置</text>
            </view>
        </view>

        <!-- 二维码区域 -->
        <view class="qrcode-section">
            <view class="qrcode-card">
                <view class="qrcode-header">
                    <text class="paw-left">🐾</text>
                    <text class="qrcode-title">扫码寻宠</text>
                    <text class="paw-right">🐾</text>
                </view>
                <view class="qrcode-wrap">
                    <image class="qrcode-img" :src="qrcodeUrl" mode="aspectFit" show-menu-by-longpress />
                    <view class="qrcode-placeholder" v-if="!qrcodeUrl">
                        <view class="placeholder-icon">
                            <text>🐕</text>
                        </view>
                        <text class="placeholder-text">二维码加载中...</text>
                    </view>
                </view>
                <view class="qrcode-tip">
                    <text class="tip-text">💝 好心人扫码可查看宠物当前位置</text>
                </view>
            </view>
        </view>

        <!-- 底部装饰 -->
        <view class="paw-decoration bottom">
            <text class="paw">🐾</text>
            <text class="paw">🐾</text>
            <text class="paw">🐾</text>
        </view>

        <!-- 底部操作按钮 -->
        <view class="bottom-actions">
            <view class="action-item" @tap="generatePoster">
                <view class="action-icon poster-icon">
                    <text>🎨</text>
                </view>
                <text class="action-text">生成海报</text>
            </view>
            <view class="action-item" @tap="saveCard">
                <view class="action-icon save-icon">
                    <text>💾</text>
                </view>
                <text class="action-text">保存卡片</text>
            </view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            // 宠物信息
            petAvatar: '',
            petName: '名宠',
            petBreed: '',
            petAge: '',
            petGender: '',
            genderIcon: '🐾',
            genderText: '萌宠',
            // 主人信息
            ownerName: '',
            ownerPhone: '',
            // 设备信息
            deviceImei: '',
            deviceStatus: 'online',
            deviceStatusText: '在线',
            // 二维码
            qrcodeUrl: ''
        };
    },
    onLoad() {
        this.loadPetData();
        this.generateQRCode();
    },
    onShow() {
        this.loadPetData();
    },
    // 分享功能
    onShareAppMessage() {
        const { petName, deviceImei } = this;
        return {
            title: `${petName}的身份卡 - 扫码查看位置`,
            path: `/pages/pet-idcard/pet-idcard?imei=${deviceImei}`,
            imageUrl: '/images/share-pet-card.png'
        };
    },
    methods: {
        // 加载宠物数据
        loadPetData() {
            const globalData = app.globalData || {};
            const selectedDevice = globalData._selectedDevice || {};

            // 从全局数据获取
            this.setData({
                petName: globalData.petName || selectedDevice.petName || '名宠',
                petAvatar: globalData.petAvatar || selectedDevice.avatar || '',
                petBreed: globalData.petBreed || selectedDevice.breed || '萌宠',
                petAge: this.calculateAge(globalData.petBirth || selectedDevice.birth),
                petGender: globalData.petGender || selectedDevice.gender || '',
                deviceImei: selectedDevice.imei || globalData.defaultDeviceImei || '862506074971204',
                ownerName: globalData.ownerName || selectedDevice.ownerName || '宠主',
                ownerPhone: globalData.ownerPhone || selectedDevice.ownerPhone || ''
            });
            this.updateGenderDisplay();
        },

        // 计算年龄
        calculateAge(birthDate) {
            if (!birthDate) {
                return '未知年龄';
            }

            // 处理 iOS 日期格式问题
            const formattedDate = birthDate.replace(/-/g, '/');
            const birth = new Date(formattedDate);
            const now = new Date();
            if (isNaN(birth.getTime())) {
                return '未知年龄';
            }
            const diffMs = now - birth;
            const diffYears = Math.floor(diffMs / (525960 * 60 * 1000));
            const diffMonths = Math.floor(diffMs / (43833.600000000006 * 60 * 1000));
            if (diffYears > 0) {
                return `${diffYears}岁`;
            } else if (diffMonths > 0) {
                return `${diffMonths}个月`;
            } else {
                return '刚出生';
            }
        },

        // 更新性别显示
        updateGenderDisplay() {
            const { petGender } = this;
            let icon = '🐾';
            let text = '萌宠';
            if (petGender === 'male' || petGender === '公') {
                icon = '♂️';
                text = '小王子';
            } else if (petGender === 'female' || petGender === '母') {
                icon = '♀️';
                text = '小公主';
            }
            this.setData({
                genderIcon: icon,
                genderText: text
            });
        },

        // 生成二维码
        generateQRCode() {
            // 生成包含宠物ID和页面路径的二维码数据
            const deviceImei = this.deviceImei;
            // 使用微信接口生成二维码
            // 实际项目中应该调用后端接口生成带参数的二维码
            // 这里使用占位图片，实际项目中替换为真实二维码URL
            this.setData({
                qrcodeUrl: '' // 后端接口返回的二维码图片URL
            });

            // 模拟延迟加载二维码
            setTimeout(() => {
                // 实际项目中这里应该是后端返回的二维码图片
                // this.setData({ qrcodeUrl: 'https://xxx.com/qrcode.png' })
            }, 500);
        },

        // 拨打主人电话
        callOwner() {
            const { ownerPhone } = this;
            if (!ownerPhone) {
                uni.showToast({
                    title: '暂无联系方式',
                    icon: 'none'
                });
                return;
            }
            uni.makePhoneCall({
                phoneNumber: ownerPhone,
                success: () => {
                    console.log('拨打电话成功');
                },
                fail: (err) => {
                    console.error('拨打电话失败', err);
                }
            });
        },

        // 查看实时位置
        viewLocation() {
            // 跳转到首页（名宠Tab）查看实时位置
            uni.switchTab({
                url: '/pages/index/index',
                success: () => {
                    // 可以在这里设置一个全局标记，让首页自动展开位置信息
                    if (app.globalData) {
                        app.globalData._autoShowLocation = true;
                    }
                }
            });
        },

        // 生成分享海报
        generatePoster() {
            uni.showLoading({
                title: '生成中...',
                mask: true
            });

            // 实际项目中这里使用 canvas 绘制海报
            // 然后保存或分享
            setTimeout(() => {
                uni.hideLoading();
                uni.showToast({
                    title: '海报功能开发中',
                    icon: 'none'
                });
            }, 500);
        },

        // 保存卡片到相册
        saveCard() {
            // 检查权限
            uni.getSetting({
                success: (res) => {
                    if (!res.authSetting['scope.writePhotosAlbum']) {
                        uni.authorize({
                            scope: 'scope.writePhotosAlbum',
                            success: () => {
                                this.doSaveCard();
                            },
                            fail: () => {
                                uni.showModal({
                                    title: '提示',
                                    content: '需要访问相册权限才能保存图片',
                                    confirmText: '去设置',
                                    success: (modalRes) => {
                                        if (modalRes.confirm) {
                                            uni.openSetting();
                                        }
                                    }
                                });
                            }
                        });
                    } else {
                        this.doSaveCard();
                    }
                }
            });
        },

        // 执行保存卡片
        doSaveCard() {
            uni.showLoading({
                title: '保存中...',
                mask: true
            });

            // 实际项目中这里需要先使用 canvas 绘制完整卡片
            // 然后保存图片到相册
            setTimeout(() => {
                uni.hideLoading();
                uni.showToast({
                    title: '保存功能开发中',
                    icon: 'none'
                });
            }, 500);
        }
    }
};
</script>
<style>
@import './pet-idcard.css';
</style>
