<template>
    <view class="page">
        <view class="list card">
            <view class="item" @tap="onSelect" data-value="公">公</view>
            <view class="item" @tap="onSelect" data-value="母">母</view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {};
    },
    methods: {
        onSelect(e) {
            const value = e.currentTarget.dataset.value;
            getApp().globalData._selectedGender = value;
            uni.navigateBack();
        }
    }
};
</script>
<style>
@import './gender-select.css';
</style>
