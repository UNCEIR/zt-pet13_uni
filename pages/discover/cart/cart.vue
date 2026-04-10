<template>
    <view class="cart-page">
        <!-- 空购物车状态 -->
        <view class="empty-cart" v-if="cartItems.length === 0">
            <image class="empty-icon" src="/static/images/empty-cart.png" mode="aspectFit"></image>
            <text class="empty-text">购物车是空的</text>
            <button class="go-shopping-btn" @tap="onGoShopping">去逛逛</button>
        </view>

        <!-- 购物车列表 -->
        <view class="cart-content" v-else>
            <!-- 购物车商品列表 -->
            <view class="cart-list">
                <view class="cart-item card" v-for="(item, index) in cartItems" :key="index">
                    <!-- 选择框 -->

                    <view :class="'checkbox ' + (item.selected ? 'checked' : '')" @tap="onSelectItem" :data-index="index">
                        <text class="check-icon" v-if="item.selected">✓</text>
                    </view>

                    <!-- 商品图片 -->

                    <image class="item-image" :src="item.image" mode="aspectFill" @tap="onViewProduct" :data-id="item.productId"></image>

                    <!-- 商品信息 -->

                    <view class="item-info">
                        <view class="item-name" @tap="onViewProduct" :data-id="item.productId">{{ item.name }}</view>
                        <view class="item-spec">{{ item.spec }}</view>
                        <view class="item-bottom">
                            <text class="item-price">¥{{ item.price }}</text>
                            <view class="quantity-control">
                                <text :class="'qty-btn ' + (item.quantity <= 1 ? 'disabled' : '')" :data-index="index" @tap="onDecreaseQuantity">-</text>
                                <text class="qty-value">{{ item.quantity }}</text>
                                <text :class="'qty-btn ' + (item.quantity >= item.stock ? 'disabled' : '')" :data-index="index" @tap="onIncreaseQuantity">+</text>
                            </view>
                        </view>
                    </view>

                    <!-- 删除按钮 -->

                    <view class="delete-btn" :data-index="index" @tap="onDeleteItem">
                        <text class="delete-icon">🗑</text>
                    </view>
                </view>
            </view>

            <!-- 猜你喜欢 -->
            <view class="recommend-section" v-if="recommendProducts.length > 0">
                <view class="recommend-title">
                    <text class="title-line"></text>
                    <text>猜你喜欢</text>
                    <text class="title-line"></text>
                </view>
                <view class="recommend-grid">
                    <view class="recommend-item" @tap="onViewProduct" :data-id="item.id" v-for="(item, index) in recommendProducts" :key="index">
                        <image class="recommend-image" :src="item.image" mode="aspectFill"></image>

                        <view class="recommend-name">{{ item.name }}</view>

                        <view class="recommend-price">¥{{ item.price }}</view>
                    </view>
                </view>
            </view>
        </view>

        <!-- 底部结算栏 -->
        <view class="bottom-bar" v-if="cartItems.length > 0">
            <view class="select-all" @tap="onSelectAll">
                <view :class="'checkbox ' + (allSelected ? 'checked' : '')">
                    <text class="check-icon" v-if="allSelected">✓</text>
                </view>
                <text class="select-text">全选</text>
            </view>
            <view class="total-section">
                <text class="total-label">合计:</text>
                <text class="total-symbol">¥</text>
                <text class="total-value">{{ totalPrice }}</text>
            </view>
            <button :class="'checkout-btn ' + (selectedCount === 0 ? 'disabled' : '')" @tap="onCheckout" :disabled="selectedCount === 0">结算({{ selectedCount }})</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            cartItems: [],
            allSelected: false,
            totalPrice: 0,
            selectedCount: 0,
            recommendProducts: []
        };
    },
    onLoad() {
        this.loadCartItems();
        this.loadRecommendProducts();
    },
    onShow() {
        this.loadCartItems();
    },
    onPullDownRefresh() {
        this.loadCartItems();
        uni.stopPullDownRefresh();
    },
    methods: {
        // 加载购物车数据
        loadCartItems() {
            // 从本地存储或服务器获取
            // 模拟数据
            const cartItems = [
                {
                    id: 'cart_001',
                    productId: 'product_001',
                    name: '进口天然猫粮 无谷物配方',
                    image: 'https://images.unsplash.com/photo-1589924691995-400dc9ecc119?w=200&h=200&fit=crop',
                    spec: '5kg装',
                    price: 128,
                    quantity: 1,
                    stock: 99,
                    selected: false
                },
                {
                    id: 'cart_002',
                    productId: 'product_002',
                    name: '宠物自动饮水机 循环过滤',
                    image: 'https://images.unsplash.com/photo-1581888227599-779811939961?w=200&h=200&fit=crop',
                    spec: '白色款',
                    price: 89,
                    quantity: 1,
                    stock: 50,
                    selected: false
                },
                {
                    id: 'cart_003',
                    productId: 'product_003',
                    name: '耐咬狗狗玩具球 发声款',
                    image: 'https://images.unsplash.com/photo-1576201836106-db17578fd166?w=200&h=200&fit=crop',
                    spec: '大号',
                    price: 29,
                    quantity: 2,
                    stock: 100,
                    selected: false
                }
            ];
            this.setData(
                {
                    cartItems
                },
                () => {
                    this.calculateTotal();
                }
            );
        },

        // 加载推荐商品
        loadRecommendProducts() {
            const recommendProducts = [
                {
                    id: 'product_004',
                    name: '宠物洗护套装',
                    price: 68,
                    image: 'https://images.unsplash.com/photo-1516734212186-a967f81ad0d7?w=400&h=400&fit=crop'
                },
                {
                    id: 'product_005',
                    name: '猫爬架玩具',
                    price: 199,
                    image: 'https://images.unsplash.com/photo-1545249390-6bdfa286032f?w=400&h=400&fit=crop'
                },
                {
                    id: 'product_006',
                    name: '宠物牵引绳',
                    price: 45,
                    image: 'https://images.unsplash.com/photo-1601758124096-1fd661873b95?w=400&h=400&fit=crop'
                },
                {
                    id: 'product_007',
                    name: '宠物窝垫',
                    price: 88,
                    image: 'https://images.unsplash.com/photo-1591382386627-349b692688ff?w=400&h=400&fit=crop'
                }
            ];
            this.setData({
                recommendProducts
            });
        },

        // 选择/取消选择商品
        onSelectItem(e) {
            const index = e.currentTarget.dataset.index;
            const key = `cartItems[${index}].selected`;
            this.setData(
                {
                    [key]: !this.cartItems[index].selected
                },
                () => {
                    this.calculateTotal();
                }
            );
        },

        // 全选/取消全选
        onSelectAll() {
            const allSelected = !this.allSelected;
            const cartItems = this.cartItems.map((item) => ({
                ...item,
                selected: allSelected
            }));
            this.setData(
                {
                    cartItems,
                    allSelected
                },
                () => {
                    this.calculateTotal();
                }
            );
        },

        // 减少数量
        onDecreaseQuantity(e) {
            const index = e.currentTarget.dataset.index;
            const item = this.cartItems[index];
            if (item.quantity <= 1) {
                return;
            }
            const key = `cartItems[${index}].quantity`;
            this.setData(
                {
                    [key]: item.quantity - 1
                },
                () => {
                    this.calculateTotal();
                    this.updateCartItem(item.id, item.quantity - 1);
                }
            );
        },

        // 增加数量
        onIncreaseQuantity(e) {
            const index = e.currentTarget.dataset.index;
            const item = this.cartItems[index];
            if (item.quantity >= item.stock) {
                uni.showToast({
                    title: '已达到库存上限',
                    icon: 'none'
                });
                return;
            }
            const key = `cartItems[${index}].quantity`;
            this.setData(
                {
                    [key]: item.quantity + 1
                },
                () => {
                    this.calculateTotal();
                    this.updateCartItem(item.id, item.quantity + 1);
                }
            );
        },

        // 删除商品
        onDeleteItem(e) {
            const index = e.currentTarget.dataset.index;
            const item = this.cartItems[index];
            uni.showModal({
                title: '确认删除',
                content: '确定要删除这个商品吗？',
                confirmColor: '#ff4757',
                success: (res) => {
                    if (res.confirm) {
                        const cartItems = this.cartItems.filter((_, i) => i !== index);
                        this.setData(
                            {
                                cartItems
                            },
                            () => {
                                this.calculateTotal();
                                this.deleteCartItem(item.id);
                            }
                        );
                        uni.showToast({
                            title: '已删除',
                            icon: 'success'
                        });
                    }
                }
            });
        },

        // 计算总价
        calculateTotal() {
            const cartItems = this.cartItems;
            const selectedItems = cartItems.filter((item) => item.selected);
            const totalPrice = selectedItems.reduce((sum, item) => {
                return sum + item.price * item.quantity;
            }, 0);
            const selectedCount = selectedItems.reduce((count, item) => {
                return count + item.quantity;
            }, 0);
            const allSelected = cartItems.length > 0 && cartItems.every((item) => item.selected);
            this.setData({
                totalPrice: totalPrice.toFixed(2),
                selectedCount,
                allSelected
            });
        },

        // 更新购物车商品数量（实际开发时调用API）
        updateCartItem(id, quantity) {
            console.log('更新购物车商品:', id, '数量:', quantity);
        },

        // 删除购物车商品（实际开发时调用API）
        deleteCartItem(id) {
            console.log('删除购物车商品:', id);
        },

        // 查看商品详情
        onViewProduct(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/product-detail?id=${id}`
            });
        },

        // 去逛逛
        onGoShopping() {
            uni.switchTab({
                url: '/pages/discover/discover'
            });
        },

        // 结算
        onCheckout() {
            const selectedItems = this.cartItems.filter((item) => item.selected);
            if (selectedItems.length === 0) {
                uni.showToast({
                    title: '请选择商品',
                    icon: 'none'
                });
                return;
            }

            // 保存选中的商品到本地存储
            uni.setStorageSync('checkoutItems', selectedItems);
            uni.navigateTo({
                url: '/pages/discover/order-confirm'
            });
        }
    }
};
</script>
<style>
@import './cart.css';
</style>
