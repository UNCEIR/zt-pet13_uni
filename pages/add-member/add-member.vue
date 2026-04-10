<template>
    <view class="page">
        <view class="tip-wrap">
            <text class="tip-text">提示：添加成功后，该成员登录小程序或APP即可进行宠物的管理</text>
        </view>

        <view class="member-card">
            <view class="member-item">
                <view class="avatar-placeholder"></view>
                <view class="member-info">
                    <text class="member-name">{{ ownerName }} (我)</text>
                    <text class="member-role">创建者</text>
                </view>
                <view class="arrow"></view>
            </view>
            <view class="divider"></view>
            <view class="member-item" @tap="goToInvite">
                <view class="invite-icon">
                    <text class="invite-plus">+</text>
                </view>
                <view class="member-info single-line">
                    <text class="member-name">邀请其他成员</text>
                </view>
                <view class="arrow"></view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            ownerName: '创建者'
        };
    },
    onShow() {
        const app = getApp();
        const ownerName = app.globalData.ownerName || uni.getStorageSync('ownerName') || '创建者';
        this.setData({
            ownerName
        });
    },
    methods: {
        goToInvite() {
            uni.navigateTo({
                url: '/pages/inviteMethod/index'
            });
        }
    }
};
</script>
<style>
@import './add-member.css';
</style>
