<template>
    <view class="page">
        <view class="sub-links card">
            <view class="sub-link" @tap="goToday">
                <image class="sub-icon" src="/static/images/icons/历史轨迹.png" mode="aspectFit" />
                <text>今日轨迹</text>
                <view class="arrow"></view>
            </view>
            <view class="sub-link" @tap="goHistory">
                <image class="sub-icon" src="/static/images/icons/历史轨迹.png" mode="aspectFit" />
                <text>历史轨迹</text>
                <view class="arrow"></view>
            </view>
        </view>
        <view class="placeholder">地图 + 时间筛选（开发中）</view>
    </view>
</template>

<script>
export default {
    data() {
        return {};
    },
    methods: {
        goToday() {
            uni.navigateTo({
                url: '/pages/track-today/track-today'
            });
        },

        goHistory() {
            uni.navigateTo({
                url: '/pages/track-history/track-history'
            });
        }
    }
};
</script>
<style>
@import './track.css';
</style>
