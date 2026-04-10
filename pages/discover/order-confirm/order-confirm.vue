<template>
    <view class="order-confirm-page">
        <!-- 收货地址 -->
        <view class="address-section card" @tap="onSelectAddress">
            <view class="address-content" v-if="address">
                <view class="address-header">
                    <text class="address-name">{{ address.name }}</text>
                    <text class="address-phone">{{ address.phone }}</text>
                </view>
                <view class="address-detail">{{ address.detail }}</view>
            </view>
            <view class="no-address" v-else>
                <text class="add-icon">+</text>
                <text>添加收货地址</text>
            </view>
            <text class="arrow">></text>
        </view>

        <!-- 订单商品 -->
        <view class="order-items card">
            <view class="section-title">商品列表</view>
            <view class="item-list">
                <view class="order-item" v-for="(item, index) in orderItems" :key="index">
                    <image class="item-image" :src="item.image" mode="aspectFill"></image>

                    <view class="item-info">
                        <view class="item-name">{{ item.name }}</view>
                        <view class="item-spec">{{ item.spec }}</view>
                        <view class="item-price-row">
                            <text class="item-price">¥{{ item.price }}</text>
                            <text class="item-quantity">x{{ item.quantity }}</text>
                        </view>
                    </view>
                </view>
            </view>
        </view>

        <!-- 配送方式 -->
        <view class="delivery-section card">
            <view class="section-title">配送方式</view>
            <picker @change="onDeliveryChange" :value="deliveryIndex" :range="deliveryMethods">
                <view class="picker-item">
                    <text>{{ deliveryMethods[deliveryIndex] }}</text>
                    <text class="arrow">></text>
                </view>
            </picker>
        </view>

        <!-- 优惠券 -->
        <view class="coupon-section card" @tap="onSelectCoupon">
            <view class="section-title">优惠券</view>
            <view class="coupon-value">
                <text class="coupon-amount" v-if="selectedCoupon">-¥{{ selectedCoupon.amount }}</text>
                <text class="coupon-name" v-if="selectedCoupon">{{ selectedCoupon.name }}</text>
                <text class="no-coupon" v-else>无可用优惠券</text>
                <text class="arrow">></text>
            </view>
        </view>

        <!-- 订单备注 -->
        <view class="remark-section card">
            <view class="section-title">订单备注</view>
            <textarea class="remark-input" placeholder="请输入备注信息，如配送时间要求等" :value="remark" @input="onRemarkInput" maxlength="100"></textarea>
        </view>

        <!-- 价格明细 -->
        <view class="price-section card">
            <view class="price-row">
                <text class="price-label">商品总价</text>
                <text class="price-value">¥{{ goodsTotal }}</text>
            </view>
            <view class="price-row">
                <text class="price-label">运费</text>
                <text :class="'price-value ' + (shippingFee === 0 ? 'free' : '')">{{ shippingFee === 0 ? '免运费' : '¥' + shippingFee }}</text>
            </view>
            <view class="price-row discount" v-if="selectedCoupon">
                <text class="price-label">优惠券</text>
                <text class="price-value discount">-¥{{ selectedCoupon.amount }}</text>
            </view>
            <view class="total-row">
                <text class="total-label">实付金额</text>
                <view class="total-value">
                    <text class="total-symbol">¥</text>
                    <text class="total-number">{{ finalTotal }}</text>
                </view>
            </view>
        </view>

        <!-- 底部提交栏 -->
        <view class="bottom-bar">
            <view class="total-section">
                <text class="total-label">合计:</text>
                <text class="total-symbol">¥</text>
                <text class="total-value">{{ finalTotal }}</text>
            </view>
            <button class="submit-btn" @tap="onSubmit">提交订单</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            address: {
                name: '张三',
                phone: '138****8888',
                detail: '北京市朝阳区某某街道某某小区 1号楼 2单元 301室'
            },
            orderItems: [
                {
                    id: 'item_001',
                    name: '进口天然猫粮 无谷物配方',
                    image: 'https://images.unsplash.com/photo-1589924691995-400dc9ecc119?w=200&h=200&fit=crop',
                    price: 128,
                    spec: '5kg装',
                    quantity: 1
                },
                {
                    id: 'item_002',
                    name: '宠物自动饮水机 循环过滤',
                    image: 'https://images.unsplash.com/photo-1581888227599-779811939961?w=200&h=200&fit=crop',
                    price: 89,
                    spec: '白色款',
                    quantity: 1
                }
            ],
            deliveryMethods: ['快递配送', '自提'],
            deliveryIndex: 0,
            selectedCoupon: {
                id: 'coupon_001',
                amount: 10,
                name: '新人优惠券'
            },
            remark: '',
            goodsTotal: 217,
            shippingFee: 0,
            finalTotal: 207
        };
    },
    onLoad() {
        this.calculateTotal();
    },
    methods: {
        calculateTotal() {
            const goodsTotal = this.orderItems.reduce((sum, item) => sum + item.price * item.quantity, 0);
            const shippingFee = goodsTotal >= 99 ? 0 : 10;
            const discount = this.selectedCoupon ? this.selectedCoupon.amount : 0;
            const finalTotal = goodsTotal + shippingFee - discount;
            this.setData({
                goodsTotal,
                shippingFee,
                finalTotal: Math.max(0, finalTotal)
            });
        },

        onSelectAddress() {
            uni.showToast({
                title: '选择地址',
                icon: 'none'
            });
        },

        onDeliveryChange(e) {
            this.setData({
                deliveryIndex: e.detail.value
            });
        },

        onSelectCoupon() {
            uni.showToast({
                title: '选择优惠券',
                icon: 'none'
            });
        },

        onRemarkInput(e) {
            this.setData({
                remark: e.detail.value
            });
        },

        onSubmit() {
            if (!this.address) {
                uni.showToast({
                    title: '请选择收货地址',
                    icon: 'none'
                });
                return;
            }
            uni.showLoading({
                title: '提交中...'
            });
            setTimeout(() => {
                uni.hideLoading();
                uni.showToast({
                    title: '订单提交成功',
                    icon: 'success'
                });
                setTimeout(() => {
                    uni.redirectTo({
                        url: '/pages/discover/order-detail?id=order_123'
                    });
                }, 1500);
            }, 2000);
        }
    }
};
</script>
<style>
@import './order-confirm.css';
</style>
