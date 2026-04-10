<template>
    <view class="page">
        <!-- 顶部区域 - 宠物插画 + 版本信息 -->
        <view class="version-header">
            <view class="pet-illustration">
                <view class="pet-head">
                    <view class="pet-ear pet-ear--left"></view>
                    <view class="pet-ear pet-ear--right"></view>
                    <view class="pet-face">
                        <view class="pet-eye pet-eye--left"></view>
                        <view class="pet-eye pet-eye--right"></view>
                        <view class="pet-nose"></view>
                        <view class="pet-mouth"></view>
                    </view>
                </view>
                <view class="pet-body"></view>
            </view>
            <view class="version-brand">
                <text class="app-name">{{ versionInfo.appName }}</text>
                <view class="version-tag">
                    <text class="version-label">V</text>
                    <text class="version-number">{{ versionInfo.version }}</text>
                </view>
            </view>
        </view>

        <!-- 版本信息卡片 -->
        <view class="info-card">
            <view class="info-row">
                <text class="info-label">当前版本</text>
                <text class="info-value">{{ versionInfo.version }}</text>
            </view>
            <view class="info-divider"></view>
            <view class="info-row">
                <text class="info-label">构建日期</text>
                <text class="info-value">{{ versionInfo.buildDate }}</text>
            </view>
            <view class="info-divider"></view>
            <view class="info-row">
                <text class="info-label">应用名称</text>
                <text class="info-value">{{ versionInfo.appName }}</text>
            </view>
        </view>

        <!-- 检查更新按钮 -->
        <view class="update-section">
            <view :class="'update-btn ' + (isChecking ? 'update-btn--checking' : '') + ' ' + (updateAvailable ? 'update-btn--available' : '')" @tap="onCheckUpdate">
                <view class="btn-icon" v-if="!isChecking && !updateAvailable">
                    <view class="icon-refresh"></view>
                </view>
                <view class="btn-icon btn-icon--spin" v-if="isChecking">
                    <view class="icon-refresh"></view>
                </view>
                <view class="btn-icon" v-if="updateAvailable">
                    <view class="icon-download"></view>
                </view>
                <text class="btn-text">{{ checkBtnTextFun }}</text>
            </view>
        </view>

        <!-- 更新日志折叠面板 -->
        <view class="changelog-section">
            <view class="section-title">
                <view class="title-icon">
                    <view class="icon-log"></view>
                </view>
                <text class="title-text">更新日志</text>
                <view :class="'title-arrow ' + (showChangelog ? 'title-arrow--up' : '')" @tap="onToggleChangelog">
                    <view class="arrow-icon"></view>
                </view>
            </view>

            <view class="changelog-content" v-if="showChangelog">
                <view class="changelog-item" v-for="(item, index) in changelog" :key="index">
                    <view class="changelog-header">
                        <text class="changelog-version">v{{ item.version }}</text>
                        <text class="changelog-date">{{ item.date }}</text>
                    </view>

                    <view class="changelog-list">
                        <view class="changelog-line" v-for="(line, index1) in item.items" :key="index1">
                            <view class="line-dot"></view>

                            <text class="line-text">{{ line }}</text>
                        </view>
                    </view>
                </view>
            </view>
        </view>

        <!-- 法律条款区 -->
        <view class="legal-section">
            <view class="section-title">
                <view class="title-icon">
                    <view class="icon-shield"></view>
                </view>
                <text class="title-text">法律条款</text>
            </view>
            <view class="legal-list">
                <view class="legal-item" @tap="onPrivacyPolicy">
                    <text class="legal-text">隐私政策</text>
                    <view class="legal-arrow"></view>
                </view>
                <view class="legal-divider"></view>
                <view class="legal-item" @tap="onUserAgreement">
                    <text class="legal-text">用户协议</text>
                    <view class="legal-arrow"></view>
                </view>
                <view class="legal-divider"></view>
                <view class="legal-item" @tap="onOpenSource">
                    <text class="legal-text">开源许可</text>
                    <view class="legal-arrow"></view>
                </view>
            </view>
        </view>

        <!-- 底部版权信息 -->
        <view class="version-footer">
            <view class="footer-decoration">
                <view class="paw-line">
                    <view class="paw-item"></view>
                    <view class="paw-item"></view>
                    <view class="paw-item"></view>
                </view>
            </view>
            <text class="copyright">© {{ currentYear }} 名宠科技</text>
            <text class="copyright-sub">PetCare Technology Co., Ltd.</text>
            <text class="footer-tip">All rights reserved</text>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            versionInfo: {
                version: '1.2.3',
                buildDate: '2024-03-11',
                appName: '名宠定位'
            },

            changelog: [
                {
                    version: '1.2.3',
                    date: '2024-03-01',
                    items: ['新增亲情号功能', '优化定位精度', '修复已知问题']
                },
                {
                    version: '1.2.0',
                    date: '2024-02-15',
                    items: ['新增电子围栏功能', 'UI界面全面优化', '提升应用稳定性']
                },
                {
                    version: '1.1.5',
                    date: '2024-01-20',
                    items: ['新增健康监测模块', '优化电池续航显示', '改进轨迹回放体验']
                }
            ],

            showChangelog: false,
            isChecking: false,
            updateAvailable: false,
            currentYear: new Date().getFullYear(),
            checkBtnText: false,
            line: '',
            checkBtnTextFun: false
        };
    },
    onLoad() {
        this.loadVersionInfo();
    },
    methods: {
        // 加载版本信息
        loadVersionInfo() {
            // 可以从本地存储或服务器获取最新版本信息
            const savedVersion = uni.getStorageSync('app_version_info');
            if (savedVersion) {
                this.setData({
                    'versionInfo.version': savedVersion.version || this.versionInfo.version,
                    'versionInfo.buildDate': savedVersion.buildDate || this.versionInfo.buildDate
                });
            }
        },

        // 切换更新日志显示
        onToggleChangelog() {
            this.setData({
                showChangelog: !this.showChangelog
            });
        },

        // 检查更新
        onCheckUpdate() {
            if (this.isChecking) {
                return;
            }

            // 如果已经有更新可用，提示用户下载
            if (this.updateAvailable) {
                uni.showModal({
                    title: '发现新版本',
                    content: '新版本 v1.3.0 已准备好，是否立即下载？',
                    confirmText: '立即下载',
                    confirmColor: '#89E04E',
                    cancelText: '稍后再说',
                    success: (res) => {
                        if (res.confirm) {
                            uni.showToast({
                                title: '开始下载...',
                                icon: 'loading',
                                duration: 2000
                            });
                        }
                    }
                });
                return;
            }

            // 开始检查更新
            this.setData({
                isChecking: true
            });

            // 模拟网络请求检查更新
            setTimeout(() => {
                // 模拟检查结果（90%概率是最新版本，10%概率有更新）
                const hasUpdate = Math.random() > 0.9;
                if (hasUpdate) {
                    this.setData({
                        isChecking: false,
                        updateAvailable: true
                    });
                    uni.showModal({
                        title: '发现新版本',
                        content: '检测到新版本 v1.3.0，包含以下更新：\n\n• 全新宠物相册功能\n• 优化定位响应速度\n• 修复部分设备兼容性问题',
                        confirmText: '立即更新',
                        confirmColor: '#89E04E',
                        cancelText: '稍后再说',
                        success: (res) => {
                            if (res.confirm) {
                                uni.showToast({
                                    title: '开始下载...',
                                    icon: 'loading'
                                });
                            } else {
                                // 用户选择稍后，保持 updateAvailable 状态
                            }
                        }
                    });
                } else {
                    this.setData({
                        isChecking: false
                    });
                    uni.showToast({
                        title: '已是最新版本',
                        icon: 'success',
                        duration: 2000
                    });
                }
            }, 1500);
        },

        // 隐私政策
        onPrivacyPolicy() {
            uni.navigateTo({
                url: '/pages/privacy-policy/privacy-policy'
            });
        },

        // 用户协议
        onUserAgreement() {
            uni.navigateTo({
                url: '/pages/user-agreement/user-agreement'
            });
        },

        // 开源许可
        onOpenSource() {
            uni.showModal({
                title: '开源许可',
                content: '本应用使用了以下开源项目：\n\n• WeChat Mini Program Framework\n• Tencent CloudBase\n• 以及其他开源组件\n\n感谢开源社区的贡献！',
                showCancel: false,
                confirmText: '知道了',
                confirmColor: '#89E04E'
            });
        },

        // 计算属性：按钮文字
        checkBtnTextFun() {
            const { isChecking, updateAvailable } = this;
            if (isChecking) {
                return '检查中...';
            }
            if (updateAvailable) {
                return '立即下载';
            }
            return '检查更新';
        }
    }
};
</script>
<style>
@import './version.css';
</style>
