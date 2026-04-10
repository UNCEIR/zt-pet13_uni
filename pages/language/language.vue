<template>
    <view class="page">
        <!-- 顶部宠物插画区域 -->
        <view class="lang-header">
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
                <view class="pet-paw pet-paw--left">
                    <view class="paw-pad"></view>
                </view>
                <view class="pet-paw pet-paw--right">
                    <view class="paw-pad"></view>
                </view>
            </view>
            <text class="lang-title">选择语言</text>
            <text class="lang-subtitle">Select Language / 言語を選択</text>
        </view>

        <!-- 语言选项卡片 -->
        <view class="lang-options">
            <view
                :class="'lang-card ' + (item.selected ? 'lang-card--selected' : '')"
                :data-code="item.code"
                @tap="onLanguageSelect"
                v-for="(item, index) in languages"
                :key="index"
            >
                <view class="lang-flag">{{ item.flag }}</view>

                <view class="lang-info">
                    <text class="lang-name">{{ item.name }}</text>
                    <text class="lang-subname">{{ item.subname }}</text>
                </view>

                <view class="lang-check" v-if="item.selected">
                    <view class="check-icon"></view>
                </view>
            </view>
        </view>

        <!-- 底部提示 -->
        <view class="lang-footer">
            <view class="pet-paw-small"></view>
            <text class="lang-tip">语言切换将立即生效</text>
            <text class="lang-tip-sub">Language will be changed immediately</text>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            languages: [
                {
                    code: 'zh-CN',
                    name: '中文',
                    subname: '简体',
                    flag: '🇨🇳',
                    selected: true
                },
                {
                    code: 'en',
                    name: 'English',
                    subname: 'English',
                    flag: '🇬🇧',
                    selected: false
                },
                {
                    code: 'ja',
                    name: '日本語',
                    subname: 'Japanese',
                    flag: '🇯🇵',
                    selected: false
                }
            ],
            currentLang: 'zh-CN'
        };
    },
    onLoad() {
        this.loadCurrentLanguage();
    },
    onShow() {
        this.loadCurrentLanguage();
    },
    // 下拉刷新
    onPullDownRefresh() {
        this.loadCurrentLanguage();
        uni.stopPullDownRefresh();
    },
    methods: {
        // 加载当前语言设置
        loadCurrentLanguage() {
            const savedLang = uni.getStorageSync('app_language') || 'zh-CN';
            this.updateLanguageSelection(savedLang);
        },

        // 更新语言选中状态
        updateLanguageSelection(langCode) {
            const languages = this.languages.map((item) => ({
                ...item,
                selected: item.code === langCode
            }));
            this.setData({
                languages,
                currentLang: langCode
            });
        },

        // 选择语言
        onLanguageSelect(e) {
            const { code } = e.currentTarget.dataset;
            const selectedLang = this.languages.find((item) => item.code === code);
            if (!selectedLang || selectedLang.selected) {
                return;
            }

            // 更新选中状态
            this.updateLanguageSelection(code);

            // 保存到本地存储
            uni.setStorageSync('app_language', code);

            // 触发全局语言切换事件
            if (app.globalData) {
                app.globalData.currentLanguage = code;
            }

            // 显示切换成功提示
            uni.showToast({
                title: this.getToastMessage(code),
                icon: 'success',
                duration: 1500
            });

            // 延迟返回上一页
            setTimeout(() => {
                uni.navigateBack();
            }, 1600);
        },

        // 获取切换成功提示消息
        getToastMessage(code) {
            const messages = {
                'zh-CN': '切换成功',
                en: 'Changed',
                ja: '変更完了'
            };
            return messages[code] || '切换成功';
        }
    }
};
</script>
<style>
@import './language.css';
</style>
