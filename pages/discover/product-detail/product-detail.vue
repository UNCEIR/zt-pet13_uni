<template>
    <view class="product-detail-page">
        <!-- 商品轮播图 -->
        <view class="gallery-section">
            <swiper class="gallery-swiper" :indicator-dots="true" indicator-color="rgba(255,255,255,0.5)" indicator-active-color="#ffffff">
                <swiper-item v-for="(item, index) in product.images" :key="index">
                    <image class="gallery-image" :src="item" mode="aspectFill" @tap="onPreviewImage" :data-index="index"></image>
                </swiper-item>
            </swiper>
        </view>

        <!-- 商品基本信息 -->
        <view class="product-info card">
            <view class="price-row">
                <text class="price-symbol">¥</text>
                <text class="price-value">{{ product.price }}</text>
                <text class="original-price" v-if="product.originalPrice">¥{{ product.originalPrice }}</text>
                <view class="tags">
                    <text class="tag discount" v-if="product.originalPrice">省{{ discountPercent }}%</text>
                    <text class="tag hot" v-if="product.isHot">热销</text>
                </view>
            </view>
            <view class="product-name">{{ product.name }}</view>
            <view class="product-desc">{{ product.description }}</view>
            <view class="sales-row">
                <text>销量 {{ product.sales }}</text>
                <text class="divider">|</text>
                <text>{{ product.likes }}人喜欢</text>
            </view>
        </view>

        <!-- 规格选择 -->
        <view class="spec-section card" @tap="onSelectSpec">
            <view class="section-label">选择规格</view>
            <view class="spec-value">
                <text v-if="selectedSpec">{{ selectedSpec.name }}</text>
                <text class="placeholder" v-else>请选择规格</text>
            </view>
            <text class="arrow">></text>
        </view>

        <!-- 商品详情 -->
        <view class="detail-section card">
            <view class="section-title">商品详情</view>
            <view class="detail-images">
                <image :src="item" mode="widthFix" :lazy-load="true" v-for="(item, index) in product.detailImages" :key="index"></image>
            </view>
        </view>

        <!-- 用户评价 -->
        <view class="reviews-section card">
            <view class="section-header">
                <view class="section-title">用户评价 ({{ product.reviewCount }})</view>
                <view class="rating">
                    <text class="rating-value">{{ product.rating }}</text>
                    <text class="rating-total">/5</text>
                </view>
            </view>
            <view class="review-list">
                <view class="review-item" v-for="(item, index) in reviews" :key="index">
                    <view class="review-header">
                        <image class="reviewer-avatar" :src="item.user.avatar"></image>
                        <view class="reviewer-info">
                            <text class="reviewer-name">{{ item.user.name }}</text>
                            <view class="rating-stars">
                                <text :class="'star ' + (star <= item.rating ? 'active' : '')" v-for="(star, index1) in [1, 2, 3, 4, 5]" :key="index1">★</text>
                            </view>
                        </view>
                        <text class="review-date">{{ item.date }}</text>
                    </view>

                    <view class="review-content">{{ item.content }}</view>

                    <view class="review-images" v-if="item.images && item.images.length > 0">
                        <image :src="img" mode="aspectFill" v-for="(img, index1) in item.images" :key="index1"></image>
                    </view>
                </view>
            </view>
            <view class="view-all-reviews" @tap="onViewAllReviews">查看全部评价 ></view>
        </view>

        <!-- 推荐商品 -->
        <view class="recommend-section">
            <view class="section-title">猜你喜欢</view>
            <view class="recommend-grid">
                <view class="recommend-item" @tap="onRecommendTap" :data-id="item.id" v-for="(item, index) in recommendProducts" :key="index">
                    <image class="recommend-image" :src="item.image" mode="aspectFill"></image>

                    <view class="recommend-name">{{ item.name }}</view>

                    <view class="recommend-price">¥{{ item.price }}</view>
                </view>
            </view>
        </view>

        <!-- 底部操作栏 -->
        <view class="bottom-bar">
            <view class="left-actions">
                <view class="action-btn" @tap="onGoHome">
                    <text class="action-icon">🏠</text>
                    <text class="action-text">首页</text>
                </view>
                <view class="action-btn" @tap="onContactService">
                    <text class="action-icon">💬</text>
                    <text class="action-text">客服</text>
                </view>
                <view class="action-btn cart-btn" @tap="onGoCart">
                    <text class="action-icon">🛒</text>
                    <text class="action-text">购物车</text>
                    <view class="cart-badge" v-if="cartCount > 0">{{ cartCount }}</view>
                </view>
            </view>
            <view class="right-actions">
                <button class="btn-add-cart" @tap="onAddCart">加入购物车</button>
                <button class="btn-buy" @tap="onBuyNow">立即购买</button>
            </view>
        </view>

        <!-- 规格选择弹窗 -->
        <view :class="'spec-popup ' + (showSpecPopup ? 'show' : '')" @tap="onCloseSpecPopup">
            <view class="popup-content" @tap.stop.prevent="onPopupContentTap">
                <view class="popup-header">
                    <image class="popup-image" :src="product.images[0]" mode="aspectFill"></image>
                    <view class="popup-info">
                        <view class="popup-price">¥{{ selectedSpec ? selectedSpec.price : product.price }}</view>
                        <view class="popup-stock">库存 {{ selectedSpec ? selectedSpec.stock : product.stock }}件</view>
                        <view class="popup-selected" v-if="selectedSpec">已选：{{ selectedSpec.name }}</view>
                    </view>
                    <view class="popup-close" @tap="onCloseSpecPopup">×</view>
                </view>
                <view class="popup-body">
                    <view class="spec-group">
                        <view class="spec-group-title">规格</view>
                        <view class="spec-options">
                            <view
                                :class="'spec-option ' + (selectedSpec && selectedSpec.id === item.id ? 'active' : '') + ' ' + (item.stock === 0 ? 'disabled' : '')"
                                :data-spec="item"
                                @tap="onSpecSelect"
                                v-for="(item, index) in product.specs"
                                :key="index"
                            >
                                {{ item.name }}
                            </view>
                        </view>
                    </view>
                    <view class="quantity-row">
                        <text class="quantity-label">数量</text>
                        <view class="quantity-control">
                            <text :class="'quantity-btn ' + (quantity <= 1 ? 'disabled' : '')" @tap="onDecreaseQuantity">-</text>
                            <text class="quantity-value">{{ quantity }}</text>
                            <text :class="'quantity-btn ' + (quantity >= maxQuantity ? 'disabled' : '')" @tap="onIncreaseQuantity">+</text>
                        </view>
                    </view>
                </view>
                <view class="popup-footer">
                    <button class="popup-btn btn-add-cart" @tap="onConfirmAddCart">加入购物车</button>
                    <button class="popup-btn btn-buy" @tap="onConfirmBuy">立即购买</button>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            productId: '',

            product: {
                id: '',
                name: '',
                price: 0,
                originalPrice: 0,
                description: '',
                images: [],
                detailImages: [],
                sales: 0,
                likes: 0,
                rating: 5,
                reviewCount: 0,
                stock: 0,
                isHot: false,
                specs: []
            },

            selectedSpec: null,
            quantity: 1,
            maxQuantity: 99,
            showSpecPopup: false,
            cartCount: 0,
            reviews: [],
            recommendProducts: [],
            discountPercent: '',
            star: 0,
            img: ''
        };
    },
    onLoad(options) {
        const { id } = options;
        if (!id) {
            uni.showToast({
                title: '商品ID不存在',
                icon: 'none'
            });
            uni.navigateBack();
            return;
        }
        this.setData({
            productId: id
        });
        this.loadProductDetail(id);
        this.loadReviews(id);
        this.loadRecommendProducts();
        this.loadCartCount();
    },
    // 分享
    onShareAppMessage() {
        const { product } = this;
        return {
            title: product.name,
            path: `/pages/discover/product-detail?id=${product.id}`,
            imageUrl: product.images[0]
        };
    },
    methods: {
        // 加载商品详情
        loadProductDetail(id) {
            // 模拟数据
            const product = {
                id: id,
                name: '进口天然猫粮 无谷物配方',
                price: 128,
                originalPrice: 158,
                description: '选用优质天然食材，无谷物配方，易消化吸收，富含Omega-3脂肪酸，有助于美毛护肤。',
                images: [
                    'https://images.unsplash.com/photo-1589924691995-400dc9ecc119?w=800&h=800&fit=crop',
                    'https://images.unsplash.com/photo-1545249390-6bdfa286032f?w=800&h=800&fit=crop',
                    'https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=800&h=800&fit=crop'
                ],
                detailImages: [
                    'https://images.unsplash.com/photo-1589924691995-400dc9ecc119?w=750&h=1000&fit=crop',
                    'https://images.unsplash.com/photo-1545249390-6bdfa286032f?w=750&h=1000&fit=crop'
                ],
                sales: 1234,
                likes: 567,
                rating: 4.8,
                reviewCount: 256,
                stock: 99,
                isHot: true,
                specs: [
                    {
                        id: 'spec_001',
                        name: '1kg装',
                        price: 128,
                        stock: 50
                    },
                    {
                        id: 'spec_002',
                        name: '5kg装',
                        price: 568,
                        stock: 30
                    },
                    {
                        id: 'spec_003',
                        name: '10kg装',
                        price: 1080,
                        stock: 20
                    }
                ]
            };
            const discountPercent = product.originalPrice ? Math.round((1 - product.price / product.originalPrice) * 100) : 0;
            this.setData({
                product,
                discountPercent,
                selectedSpec: product.specs[0],
                maxQuantity: product.specs[0].stock
            });
        },

        // 加载评价
        loadReviews(id) {
            // 模拟数据
            const reviews = [
                {
                    id: 'review_001',
                    user: {
                        name: '豆豆麻麻',
                        avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                    },
                    rating: 5,
                    content: '我家猫咪很喜欢吃，毛色也变亮了，会回购！',
                    date: '2026-03-10',
                    images: []
                },
                {
                    id: 'review_002',
                    user: {
                        name: '咪咪主人',
                        avatar: 'https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=100&h=100&fit=crop'
                    },
                    rating: 5,
                    content: '包装很好，物流也很快，猫咪吃得香。',
                    date: '2026-03-08',
                    images: ['https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=400&h=400&fit=crop']
                },
                {
                    id: 'review_003',
                    user: {
                        name: '旺财粑粑',
                        avatar: 'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=100&h=100&fit=crop'
                    },
                    rating: 4,
                    content: '质量不错，就是价格有点小贵。',
                    date: '2026-03-05',
                    images: []
                }
            ];
            this.setData({
                reviews
            });
        },

        // 加载推荐商品
        loadRecommendProducts() {
            // 模拟数据
            const recommendProducts = [
                {
                    id: 'product_002',
                    name: '宠物自动饮水机',
                    price: 89,
                    image: 'https://images.unsplash.com/photo-1581888227599-779811939961?w=400&h=400&fit=crop'
                },
                {
                    id: 'product_003',
                    name: '耐咬狗狗玩具球',
                    price: 29,
                    image: 'https://images.unsplash.com/photo-1576201836106-db17578fd166?w=400&h=400&fit=crop'
                },
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
                }
            ];
            this.setData({
                recommendProducts
            });
        },

        // 加载购物车数量
        loadCartCount() {
            // 从本地存储或服务器获取
            this.setData({
                cartCount: 3
            });
        },

        // 预览轮播图
        onPreviewImage(e) {
            const index = e.currentTarget.dataset.index;
            uni.previewImage({
                current: this.product.images[index],
                urls: this.product.images
            });
        },

        // 选择规格
        onSelectSpec() {
            this.setData({
                showSpecPopup: true,
                quantity: 1
            });
        },

        // 关闭规格弹窗
        onCloseSpecPopup() {
            this.setData({
                showSpecPopup: false
            });
        },

        // 阻止事件冒泡
        onPopupContentTap(e) {
            e.stopPropagation();
        },

        // 选择规格
        onSpecSelect(e) {
            const spec = e.currentTarget.dataset.spec;
            if (spec.stock === 0) {
                return;
            }
            this.setData({
                selectedSpec: spec,
                maxQuantity: spec.stock,
                quantity: 1
            });
        },

        // 减少数量
        onDecreaseQuantity() {
            if (this.quantity > 1) {
                this.setData({
                    quantity: this.quantity - 1
                });
            }
        },

        // 增加数量
        onIncreaseQuantity() {
            if (this.quantity < this.maxQuantity) {
                this.setData({
                    quantity: this.quantity + 1
                });
            }
        },

        // 确认加入购物车
        onConfirmAddCart() {
            this.addToCart();
            this.setData({
                showSpecPopup: false
            });
        },

        // 确认购买
        onConfirmBuy() {
            const { product, selectedSpec, quantity } = this;
            const item = {
                id: product.id,
                name: product.name,
                image: product.images[0],
                price: selectedSpec ? selectedSpec.price : product.price,
                spec: selectedSpec ? selectedSpec.name : '默认规格',
                quantity: quantity
            };

            // 保存到待结算订单
            uni.setStorageSync('checkoutItems', [item]);
            this.setData({
                showSpecPopup: false
            });
            uni.navigateTo({
                url: '/pages/discover/order-confirm'
            });
        },

        // 加入购物车
        onAddCart() {
            if (!this.selectedSpec) {
                this.setData({
                    showSpecPopup: true
                });
                return;
            }
            this.addToCart();
        },

        addToCart() {
            const { product, selectedSpec, quantity, cartCount } = this;

            // 实际开发时调用API或更新本地存储
            uni.showToast({
                title: '已加入购物车',
                icon: 'success'
            });
            this.setData({
                cartCount: cartCount + quantity
            });
        },

        // 立即购买
        onBuyNow() {
            if (!this.selectedSpec) {
                this.setData({
                    showSpecPopup: true
                });
                return;
            }
            const { product, selectedSpec, quantity } = this;
            const item = {
                id: product.id,
                name: product.name,
                image: product.images[0],
                price: selectedSpec.price,
                spec: selectedSpec.name,
                quantity: quantity
            };

            // 保存到待结算订单
            uni.setStorageSync('checkoutItems', [item]);
            uni.navigateTo({
                url: '/pages/discover/order-confirm'
            });
        },

        // 返回首页
        onGoHome() {
            uni.switchTab({
                url: '/pages/index/index'
            });
        },

        // 联系客服
        onContactService() {
            uni.showModal({
                title: '联系客服',
                content: '客服电话：400-888-8888\n工作时间：9:00-21:00',
                showCancel: true,
                confirmText: '拨打',
                success: (res) => {
                    if (res.confirm) {
                        uni.makePhoneCall({
                            phoneNumber: '4008888888'
                        });
                    }
                }
            });
        },

        // 去购物车
        onGoCart() {
            uni.navigateTo({
                url: '/pages/discover/cart'
            });
        },

        // 查看全部评价
        onViewAllReviews() {
            uni.navigateTo({
                url: `/pages/discover/product-reviews?productId=${this.productId}`
            });
        },

        // 点击推荐商品
        onRecommendTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/product-detail?id=${id}`
            });
        }
    }
};
</script>
<style>
@import './product-detail.css';
</style>
