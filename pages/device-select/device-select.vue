<template>
    <view class="page">
        <view class="list card">
            <view class="item" @tap="onSelect" :data-device="item" v-for="(item, index) in devices" :key="index">
                <text class="imei">{{ item.imei }}</text>

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
            devices: []
        };
    },
    onLoad() {
        this.setData({
            devices: app.globalData.devices || [
                {
                    imei: '862506074976309',
                    model: 'PT13'
                }
            ]
        });
    },
    methods: {
        onSelect(e) {
            const device = e.currentTarget.dataset.device;
            app.globalData._selectedDevice = device;
            uni.navigateBack();
        }
    }
};
</script>
<style>
@import './device-select.css';
</style>
