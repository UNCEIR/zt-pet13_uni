<template>
    <view class="page">
        <view class="list card">
            <view class="item" @tap="onSelect" :data-value="item" v-for="(item, index) in breeds" :key="index">
                {{ item }}
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            breeds: ['美短猫', '英短猫', '布偶猫', '暹罗猫', '柯基犬', '金毛', '拉布拉多', '哈士奇', '泰迪', '博美', '其他']
        };
    },
    methods: {
        onSelect(e) {
            const value = e.currentTarget.dataset.value;
            getApp().globalData._selectedBreed = value;
            uni.navigateBack();
        }
    }
};
</script>
<style>
@import './breed-select.css';
</style>
