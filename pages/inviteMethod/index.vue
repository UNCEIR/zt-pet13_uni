<template>
    <view class="page">
        <text class="section-title">二维码扫码邀请</text>
        <view class="qr-section">
            <view class="qr-code">
                <view class="qr-marker qr-tl"><view class="qr-core"></view></view>
                <view class="qr-marker qr-tr"><view class="qr-core"></view></view>
                <view class="qr-marker qr-bl"><view class="qr-core"></view></view>
                <view class="qr-data"></view>
            </view>
            <text class="qr-desc">使用微信扫码加入</text>
        </view>

        <text class="section-title wechat-title">微信分享邀请</text>
        <button class="wechat-card" open-type="share" @tap="onWechatInviteTap">
            <view class="wechat-icon">
                <view class="wx-bubble-l"></view>
                <view class="wx-bubble-r"></view>
            </view>
            <text class="wechat-text">微信好友</text>
            <view class="arrow"></view>
        </button>
    </view>
</template>

<script>
export default {
    data() {
        return {};
    },
    onShareAppMessage() {
        return {
            title: '邀请你一起管理宠物',
            path: '/pages/add-member/add-member?from=invite',
            imageUrl: ''
        };
    },
    onShareTimeline() {
        return {
            title: '邀请你一起管理宠物'
        };
    },
    methods: {
        onWechatInviteTap() {
            // 点击分享按钮时，微信会根据 open-type=share 拉起系统分享面板。
            console.log('wechat invite trigger');
        }
    }
};
</script>
<style>
@import './index.css';
</style>
