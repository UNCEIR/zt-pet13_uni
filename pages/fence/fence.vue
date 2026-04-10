<template>
    <view class="page">
        <view class="sub-links card">
            <view class="sub-link" @tap="goInOut">
                <image class="sub-icon" src="/static/images/icons/出入围栏.png" mode="aspectFit" />
                <text>出入围栏</text>
                <view class="arrow"></view>
            </view>
            <view class="sub-link" @tap="goHotspot">
                <image class="sub-icon" src="/static/images/icons/热点围栏.png" mode="aspectFit" />
                <text>热点围栏</text>
                <view class="arrow"></view>
            </view>
        </view>
        <view class="list card">
            <view class="empty" v-if="fences.length === 0">暂无围栏，点击下方按钮新增</view>
        </view>
        <view class="btn-wrap">
            <view class="btn-full">新增围栏</view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            fences: []
        };
    },
    methods: {
        goInOut() {
            uni.navigateTo({
                url: '/pages/fence-in-out/fence-in-out'
            });
        },

        goHotspot() {
            uni.navigateTo({
                url: '/pages/fence-hotspot/fence-hotspot'
            });
        }
    }
};
</script>
<style>
@import './fence.css';
</style>
