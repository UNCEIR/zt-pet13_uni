<template>
    <view class="page">
        <view class="actions card">
            <view class="action-item" v-if="!hasOwner" @tap="goAddOwner">
                <image class="action-icon" src="/static/images/icons/添加主人.png" mode="aspectFit" />
                <text>添加主人</text>
            </view>
            <view class="action-item" v-else @tap="goAddMember">
                <image class="action-icon" src="/static/images/icons/添加主人.png" mode="aspectFit" />
                <text>添加成员</text>
            </view>
            <view class="action-item" @tap="goAddPet">
                <image class="action-icon" src="/static/images/icons/添加萌宠.png" mode="aspectFit" />
                <text>添加萌宠</text>
            </view>
        </view>
        <view class="list card">
            <view class="item" v-for="(item, index) in pets" :key="index">
                <image class="avatar" :src="item.avatar || '/images/home-bg.jpg'" mode="aspectFill" />

                <view class="info">
                    <text class="name">{{ item.name }}</text>
                    <text class="detail">{{ item.breed }} · {{ item.gender }}</text>
                </view>

                <view class="arrow"></view>
            </view>
            <view class="empty" v-if="pets.length === 0">
                <text>暂无宠物，点击上方添加萌宠</text>
            </view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            pets: [],
            hasOwner: false
        };
    },
    onShow() {
        const hasOwner = app.globalData.getHasOwner();
        this.setData({
            pets: app.globalData.pets || [],
            hasOwner
        });
    },
    methods: {
        goAddOwner() {
            uni.navigateTo({
                url: '/pages/add-owner/add-owner'
            });
        },

        goAddMember() {
            uni.navigateTo({
                url: '/pages/add-member/add-member'
            });
        },

        goAddPet() {
            uni.navigateTo({
                url: '/pages/add-pet/add-pet'
            });
        }
    }
};
</script>
<style>
@import './pet-list.css';
</style>
