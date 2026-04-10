<template>
    <view class="discover-page">
        <!-- 顶部搜索栏 -->
        <view class="header">
            <view class="search-bar" @tap="onSearchTap">
                <text class="search-icon">🔍</text>
                <text class="search-placeholder">搜索宠友、话题、商品...</text>
            </view>
            <view class="message-btn" @tap="onMessageTap">
                <text class="message-icon">🔔</text>
                <view class="badge" v-if="unreadCount > 0">{{ unreadCount }}</view>
            </view>
        </view>

        <!-- Tab栏 -->
        <view class="tab-bar-container">
            <scroll-view class="tab-scroll" :scroll-x="true" :show-scrollbar="false">
                <view class="tab-list">
                    <view :class="'tab-item ' + (currentTab === index ? 'active' : '')" :data-index="index" @tap="onTabTap" v-for="(item, index) in tabs" :key="index">
                        <text class="tab-text">{{ item.name }}</text>

                        <view class="tab-indicator" v-if="currentTab === index"></view>
                    </view>
                </view>
            </scroll-view>
        </view>

        <!-- Tab内容区域 - Swiper联动 -->
        <swiper class="content-swiper" :current="currentTab" @change="onSwiperChange" duration="200">
            <!-- Tab 1: 附近宠友 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-content-scroll" :refresher-enabled="true" :refresher-triggered="nearbyRefreshing" @refresherrefresh="onNearbyRefresh">
                    <view class="nearby-tab">
                        <!-- 定位提示栏 -->
                        <view class="location-bar">
                            <text class="location-icon">📍</text>
                            <text class="location-text">{{ location.name || '正在定位...' }}</text>
                            <text class="location-change" @tap="onChangeLocation">切换</text>
                        </view>

                        <!-- 搜索距离筛选 -->
                        <view class="filter-section">
                            <view class="filter-title">📍 搜索距离</view>
                            <view class="filter-bar">
                                <view :class="'filter-item ' + (nearbyFilter === 'all' ? 'active' : '')" data-filter="all" @tap="onNearbyFilterTap">全部</view>
                                <view :class="'filter-item ' + (nearbyFilter === '1km' ? 'active' : '')" data-filter="1km" @tap="onNearbyFilterTap">1km内</view>
                                <view :class="'filter-item ' + (nearbyFilter === '3km' ? 'active' : '')" data-filter="3km" @tap="onNearbyFilterTap">3km内</view>
                                <view :class="'filter-item ' + (nearbyFilter === '5km' ? 'active' : '')" data-filter="5km" @tap="onNearbyFilterTap">5km内</view>
                            </view>
                            <view class="filter-hint" v-if="nearbyFilter !== 'all'">当前显示 {{ nearbyFilter }} 范围内的宠友 ({{ filteredNearbyUsers.length }} 位)</view>
                        </view>

                        <!-- 空状态提示 -->
                        <view class="empty-state" v-if="nearbyUsers.length === 0">
                            <text class="empty-icon">🔍</text>
                            <text class="empty-text">该范围内暂无宠友</text>
                            <text class="empty-hint">试试选择更大的搜索范围</text>
                        </view>

                        <!-- 用户列表 -->
                        <view class="user-list">
                            <view class="user-card" :data-id="item.id" @tap="onUserCardTap" v-for="(item, index) in nearbyUsers" :key="index">
                                <image class="user-avatar" :src="item.pet.avatar" mode="aspectFill"></image>

                                <view class="user-info">
                                    <view class="user-name-row">
                                        <text class="user-name">{{ item.pet.name }}</text>
                                        <text class="user-online" v-if="item.isOnline">●</text>
                                    </view>
                                    <view class="user-breed">{{ item.pet.breed }}</view>
                                    <view class="user-distance">
                                        <text class="distance-icon">📍</text>
                                        <text>{{ item.distanceText }}</text>
                                    </view>
                                </view>

                                <view class="user-actions">
                                    <button class="action-btn greet" :data-id="item.id" @tap.stop.prevent="onGreetTap">打招呼</button>
                                    <button class="action-btn profile" :data-id="item.id" @tap.stop.prevent="onViewProfileTap">看主页</button>
                                </view>
                            </view>
                        </view>

                        <!-- 加载更多 -->
                        <view class="load-more" v-if="nearbyHasMore" @tap="loadMoreNearby">
                            <text>加载更多</text>
                        </view>
                    </view>
                </scroll-view>
            </swiper-item>

            <!-- Tab 2: 动态广场 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-content-scroll" :refresher-enabled="true" :refresher-triggered="feedRefreshing" @refresherrefresh="onFeedRefresh">
                    <view class="feed-tab">
                        <!-- 瀑布流布局 -->
                        <view class="waterfall">
                            <view class="waterfall-column left">
                                <view class="feed-card" :data-id="item.id" @tap="onFeedCardTap" v-for="(item, index) in feedLeft" :key="index">
                                    <image class="feed-image" :src="item.images[0]" mode="widthFix" :lazy-load="true"></image>

                                    <view class="feed-content">
                                        <text class="feed-title text-ellipsis-2">{{ item.content }}</text>
                                        <view class="feed-author">
                                            <image class="author-avatar" :src="item.author.avatar"></image>
                                            <text class="author-name">{{ item.author.name }}</text>
                                        </view>
                                        <view class="feed-stats">
                                            <text class="stat-item">❤️ {{ item.likes }}</text>
                                            <text class="stat-item">💬 {{ item.comments }}</text>
                                        </view>
                                    </view>
                                </view>
                            </view>
                            <view class="waterfall-column right">
                                <view class="feed-card" :data-id="item.id" @tap="onFeedCardTap" v-for="(item, index) in feedRight" :key="index">
                                    <image class="feed-image" :src="item.images[0]" mode="widthFix" :lazy-load="true"></image>

                                    <view class="feed-content">
                                        <text class="feed-title text-ellipsis-2">{{ item.content }}</text>
                                        <view class="feed-author">
                                            <image class="author-avatar" :src="item.author.avatar"></image>
                                            <text class="author-name">{{ item.author.name }}</text>
                                        </view>
                                        <view class="feed-stats">
                                            <text class="stat-item">❤️ {{ item.likes }}</text>
                                            <text class="stat-item">💬 {{ item.comments }}</text>
                                        </view>
                                    </view>
                                </view>
                            </view>
                        </view>

                        <!-- 加载更多 -->
                        <view class="load-more" v-if="feedHasMore" @tap="loadMoreFeed">
                            <text>加载更多</text>
                        </view>
                    </view>
                </scroll-view>
            </swiper-item>

            <!-- Tab 3: 话题讨论 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-content-scroll" :refresher-enabled="true" :refresher-triggered="topicRefreshing" @refresherrefresh="onTopicRefresh">
                    <view class="topic-tab">
                        <!-- 搜索话题 -->
                        <view class="topic-search" @tap="onTopicSearchTap">
                            <text class="search-icon">🔍</text>
                            <text class="search-placeholder">搜索话题</text>
                        </view>

                        <!-- 分类标签 -->
                        <view class="topic-categories">
                            <view
                                :class="'category-item ' + (topicCategory === item.id ? 'active' : '')"
                                :data-id="item.id"
                                @tap="onTopicCategoryTap"
                                v-for="(item, index) in topicCategories"
                                :key="index"
                            >
                                <text>{{ item.name }}</text>
                            </view>
                        </view>

                        <!-- 热门话题列表 -->
                        <view class="topic-list">
                            <view class="topic-item" :data-id="item.id" @tap="onTopicTap" v-for="(item, index) in topics" :key="index">
                                <view class="topic-header">
                                    <text class="topic-tag">#{{ item.name }}</text>
                                    <text class="topic-heat">🔥 {{ item.heat }}</text>
                                </view>

                                <view class="topic-desc">{{ item.description }}</view>

                                <view class="topic-meta">
                                    <text>{{ item.participantCount }} 人参与讨论</text>
                                    <text class="topic-new" v-if="item.hasNew">有新动态</text>
                                </view>
                            </view>
                        </view>
                    </view>
                </scroll-view>
            </swiper-item>

            <!-- Tab 4: 宠物问答 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-content-scroll" :refresher-enabled="true" :refresher-triggered="qaRefreshing" @refresherrefresh="onQaRefresh">
                    <view class="qa-tab">
                        <!-- 搜索和提问 -->
                        <view class="qa-header">
                            <view class="qa-search" @tap="onQaSearchTap">
                                <text class="search-icon">🔍</text>
                                <text class="search-placeholder">搜索问题</text>
                            </view>
                            <button class="qa-ask-btn" @tap="onAskQuestionTap">+ 提问</button>
                        </view>

                        <!-- 分类标签 -->
                        <view class="qa-categories">
                            <view
                                :class="'category-item ' + (qaCategory === item.id ? 'active' : '')"
                                :data-id="item.id"
                                @tap="onQaCategoryTap"
                                v-for="(item, index) in qaCategories"
                                :key="index"
                            >
                                <text>{{ item.name }}</text>
                            </view>
                        </view>

                        <!-- 问答列表 -->
                        <view class="qa-list">
                            <view class="qa-item" :data-id="item.id" @tap="onQuestionTap" v-for="(item, index) in questions" :key="index">
                                <view class="qa-question">
                                    <text class="qa-label">Q:</text>
                                    <text class="qa-title">{{ item.title }}</text>
                                </view>

                                <view class="qa-expert" v-if="item.expertAnswer">
                                    <image class="expert-avatar" :src="item.expert.avatar"></image>
                                    <text class="expert-badge">认证专家</text>
                                    <text class="expert-name">{{ item.expert.name }} 回答</text>
                                </view>

                                <view class="qa-answer text-ellipsis-2" v-if="item.expertAnswer">{{ item.expertAnswer }}</view>

                                <view class="qa-meta">
                                    <text class="view-count">👁️ {{ item.viewCount }}</text>
                                    <text class="answer-count">{{ item.answerCount }}个回答</text>
                                    <text :class="'follow-btn ' + (item.isFollowing ? 'following' : '')" :data-id="item.id" @tap.stop.prevent="onFollowQuestionTap">
                                        {{ item.isFollowing ? '已关注' : '关注问题' }}
                                    </text>
                                </view>
                            </view>
                        </view>
                    </view>
                </scroll-view>
            </swiper-item>

            <!-- Tab 5: 线下活动 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-content-scroll" :refresher-enabled="true" :refresher-triggered="activityRefreshing" @refresherrefresh="onActivityRefresh">
                    <view class="activity-tab">
                        <!-- 搜索和发布 -->
                        <view class="activity-header">
                            <view class="activity-search" @tap="onActivitySearchTap">
                                <text class="search-icon">🔍</text>
                                <text class="search-placeholder">搜索活动</text>
                            </view>
                            <button class="activity-publish-btn" @tap="onPublishActivityTap">+ 发布</button>
                        </view>

                        <!-- 活动类型筛选 -->
                        <view class="activity-types">
                            <view
                                :class="'type-item ' + (activityType === item.id ? 'active' : '')"
                                :data-id="item.id"
                                @tap="onActivityTypeTap"
                                v-for="(item, index) in activityTypes"
                                :key="index"
                            >
                                <text>{{ item.name }}</text>
                            </view>
                        </view>

                        <!-- 活动列表 -->
                        <view class="activity-list">
                            <view class="activity-card" :data-id="item.id" @tap="onActivityTap" v-for="(item, index) in activities" :key="index">
                                <image class="activity-cover" :src="item.coverImage" mode="aspectFill"></image>

                                <view class="activity-info">
                                    <view class="activity-title text-ellipsis">{{ item.title }}</view>
                                    <view class="activity-meta">
                                        <text class="meta-item">📍 {{ item.location.name }} · {{ item.distanceText }}</text>
                                        <text class="meta-item">📅 {{ item.date }} {{ item.time }}</text>
                                        <text class="meta-item">👥 {{ item.registeredCount }}/{{ item.maxCount }}人已报名</text>
                                    </view>
                                    <button :class="'activity-join-btn ' + (item.isRegistered ? 'registered' : '')" :data-id="item.id" @tap.stop.prevent="onJoinActivityTap">
                                        {{ item.isRegistered ? '已报名' : '立即报名' }}
                                    </button>
                                </view>
                            </view>
                        </view>
                    </view>
                </scroll-view>
            </swiper-item>

            <!-- Tab 6: 好物推荐 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-content-scroll" :refresher-enabled="true" :refresher-triggered="productRefreshing" @refresherrefresh="onProductRefresh">
                    <view class="product-tab">
                        <!-- 搜索和购物车 -->
                        <view class="product-header">
                            <view class="product-search" @tap="onProductSearchTap">
                                <text class="search-icon">🔍</text>
                                <text class="search-placeholder">搜索商品</text>
                            </view>
                            <view class="cart-btn" @tap="onCartTap">
                                <text class="cart-icon">🛒</text>
                                <view class="cart-badge" v-if="cartCount > 0">{{ cartCount }}</view>
                            </view>
                        </view>

                        <!-- 商品分类 -->
                        <view class="product-categories">
                            <view
                                :class="'category-item ' + (productCategory === item.id ? 'active' : '')"
                                :data-id="item.id"
                                @tap="onProductCategoryTap"
                                v-for="(item, index) in productCategories"
                                :key="index"
                            >
                                <text>{{ item.name }}</text>
                            </view>
                        </view>

                        <!-- 商品网格 -->
                        <view class="product-grid">
                            <view class="product-card" :data-id="item.id" @tap="onProductTap" v-for="(item, index) in products" :key="index">
                                <image class="product-image" :src="item.image" mode="aspectFill" :lazy-load="true"></image>

                                <view class="product-info">
                                    <view class="product-name text-ellipsis">{{ item.name }}</view>
                                    <view class="product-price-row">
                                        <text class="product-price">¥{{ item.price }}</text>
                                        <text class="product-likes">❤️ {{ item.likes }}</text>
                                    </view>
                                    <button class="add-cart-btn" :data-id="item.id" :data-index="index" @tap.stop.prevent="onAddCartTap">加入购物车</button>
                                </view>
                            </view>
                        </view>
                    </view>
                </scroll-view>
            </swiper-item>
        </swiper>

        <!-- 底部发布悬浮按钮 -->
        <view class="fab-publish" @tap="onFabPublishTap">
            <text class="fab-icon">✏️</text>
            <text class="fab-text">发布</text>
        </view>

        <!-- 购物车悬浮栏(仅在好物推荐Tab显示) -->
        <view class="cart-float-bar" v-if="currentTab === 5 && cartCount > 0">
            <view class="cart-info">
                <text class="cart-total">¥{{ cartTotal }}</text>
                <text class="cart-count">共 {{ cartCount }} 件商品</text>
            </view>
            <button class="checkout-btn" @tap="onCheckoutTap">去结算</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            // Tab 配置
            tabs: [
                {
                    id: 'nearby',
                    name: '附近宠友'
                },
                {
                    id: 'feed',
                    name: '动态广场'
                },
                {
                    id: 'topic',
                    name: '话题讨论'
                },
                {
                    id: 'qa',
                    name: '宠物问答'
                },
                {
                    id: 'activity',
                    name: '线下活动'
                },
                {
                    id: 'product',
                    name: '好物推荐'
                }
            ],

            currentTab: 0,

            // 顶部消息
            unreadCount: 3,

            // ===== Tab 1: 附近宠友 =====
            location: {
                name: '北京市朝阳区',
                latitude: 39.9042,
                longitude: 116.4074
            },

            nearbyFilter: 'all',
            nearbyRefreshing: false,
            nearbyHasMore: true,
            allNearbyUsers: [],

            // 存储所有用户数据
            filteredNearbyUsers: [],

            // 筛选后的用户数据
            nearbyUsers: [
                {
                    id: 'user_001',
                    pet: {
                        name: '豆豆',
                        avatar: 'https://images.unsplash.com/photo-1552053831-71594a27632d?w=200&h=200&fit=crop',
                        breed: '金毛寻回犬'
                    },
                    distance: 200,
                    isOnline: true
                },
                {
                    id: 'user_002',
                    pet: {
                        name: '咪咪',
                        avatar: 'https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=200&h=200&fit=crop',
                        breed: '布偶猫'
                    },
                    distance: 450,
                    isOnline: true
                },
                {
                    id: 'user_003',
                    pet: {
                        name: '旺财',
                        avatar: 'https://images.unsplash.com/photo-1583511655857-d19b40a7a54e?w=200&h=200&fit=crop',
                        breed: '柴犬'
                    },
                    distance: 800,
                    isOnline: false
                },
                {
                    id: 'user_004',
                    pet: {
                        name: '奥利奥',
                        avatar: 'https://images.unsplash.com/photo-1587300003388-59208cc962cb?w=200&h=200&fit=crop',
                        breed: '边境牧羊犬'
                    },
                    distance: 1200,
                    isOnline: true
                },
                {
                    id: 'user_005',
                    pet: {
                        name: '花花',
                        avatar: 'https://images.unsplash.com/photo-1573865526739-10659fec78a5?w=200&h=200&fit=crop',
                        breed: '英短蓝猫'
                    },
                    distance: 1500,
                    isOnline: false
                },
                {
                    id: 'user_006',
                    pet: {
                        name: '大黄',
                        avatar: 'https://images.unsplash.com/photo-1537151608828-ea2b11777ee8?w=200&h=200&fit=crop',
                        breed: '拉布拉多'
                    },
                    distance: 1800,
                    isOnline: true
                },
                {
                    id: 'user_007',
                    pet: {
                        name: '小白',
                        avatar: 'https://images.unsplash.com/photo-1519052537078-e6302a4968d4?w=200&h=200&fit=crop',
                        breed: '萨摩耶'
                    },
                    distance: 2200,
                    isOnline: false
                },
                {
                    id: 'user_008',
                    pet: {
                        name: '可乐',
                        avatar: 'https://images.unsplash.com/photo-1543466835-00a7907e9de1?w=200&h=200&fit=crop',
                        breed: '柯基'
                    },
                    distance: 2800,
                    isOnline: true
                },
                {
                    id: 'user_009',
                    pet: {
                        name: '雪球',
                        avatar: 'https://images.unsplash.com/photo-1533738363-b7f9aef128ce?w=200&h=200&fit=crop',
                        breed: '波斯猫'
                    },
                    distance: 3500,
                    isOnline: false
                },
                {
                    id: 'user_010',
                    pet: {
                        name: '黑豹',
                        avatar: 'https://images.unsplash.com/photo-1477936432016-8172ed08637e?w=200&h=200&fit=crop',
                        breed: '德国牧羊犬'
                    },
                    distance: 4200,
                    isOnline: true
                }
            ],

            // ===== Tab 2: 动态广场 =====
            feedRefreshing: false,

            feedHasMore: true,

            feedLeft: [
                {
                    id: 'feed_001',
                    content: '今天带豆豆去公园玩，超级开心！🐕',
                    images: ['https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=400&h=500&fit=crop'],
                    author: {
                        name: '豆豆麻麻',
                        avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                    },
                    likes: 128,
                    comments: 23
                },
                {
                    id: 'feed_003',
                    content: '求助：猫咪最近不爱吃猫粮怎么办？',
                    images: ['https://images.unsplash.com/photo-1513245543132-31f507417b26?w=400&h=450&fit=crop'],
                    author: {
                        name: '咪咪主人',
                        avatar: 'https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=100&h=100&fit=crop'
                    },
                    likes: 45,
                    comments: 67
                },
                {
                    id: 'feed_005',
                    content: '边境牧羊犬真的太聪明了！教什么都一学就会',
                    images: ['https://images.unsplash.com/photo-1503256207526-0d5d80fa2f47?w=400&h=400&fit=crop'],
                    author: {
                        name: '奥利奥爸爸',
                        avatar: 'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100&h=100&fit=crop'
                    },
                    likes: 89,
                    comments: 15
                }
            ],

            feedRight: [
                {
                    id: 'feed_002',
                    content: '小柴的日常犯傻系列，笑死我了😂',
                    images: ['https://images.unsplash.com/photo-1583511655857-d19b40a7a54e?w=400&h=600&fit=crop'],
                    author: {
                        name: '旺财粑粑',
                        avatar: 'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=100&h=100&fit=crop'
                    },
                    likes: 256,
                    comments: 45
                },
                {
                    id: 'feed_004',
                    content: '周末带狗子去爬山，景色超美！',
                    images: ['https://images.unsplash.com/photo-1548199973-03cce0bbc87b?w=400&h=350&fit=crop'],
                    author: {
                        name: '豆豆麻麻',
                        avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                    },
                    likes: 78,
                    comments: 12
                },
                {
                    id: 'feed_006',
                    content: '新买的猫爬架，主子很满意～',
                    images: ['https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=400&h=480&fit=crop'],
                    author: {
                        name: '花花麻麻',
                        avatar: 'https://images.unsplash.com/photo-1544005313-94ddf0286df2?w=100&h=100&fit=crop'
                    },
                    likes: 134,
                    comments: 28
                }
            ],

            // ===== Tab 3: 话题讨论 =====
            topicRefreshing: false,

            topicCategory: 'all',

            topicCategories: [
                {
                    id: 'all',
                    name: '全部'
                },
                {
                    id: 'daily',
                    name: '日常'
                },
                {
                    id: 'help',
                    name: '求助'
                },
                {
                    id: 'photo',
                    name: '晒图'
                },
                {
                    id: 'knowledge',
                    name: '知识'
                }
            ],

            topics: [
                {
                    id: 'topic_001',
                    name: '我家宠物的奇怪习惯',
                    description: '分享你家宠物的各种奇葩习惯，看看谁家的最搞笑',
                    heat: '2.3k',
                    participantCount: 1200,
                    hasNew: true
                },
                {
                    id: 'topic_002',
                    name: '猫咪健康养护指南',
                    description: '讨论猫咪日常养护、疾病预防等健康知识',
                    heat: '1.8k',
                    participantCount: 890,
                    hasNew: false
                },
                {
                    id: 'topic_003',
                    name: '狗狗训练心得交流',
                    description: '分享训练狗狗的经验和技巧，新手必看',
                    heat: '3.1k',
                    participantCount: 2100,
                    hasNew: true
                },
                {
                    id: 'topic_004',
                    name: '宠物美容造型秀',
                    description: '晒出你家宠物的最美容颜，一起来欣赏',
                    heat: '956',
                    participantCount: 456,
                    hasNew: false
                },
                {
                    id: 'topic_005',
                    name: '领养代替购买',
                    description: '分享领养故事，传递爱心正能量',
                    heat: '1.2k',
                    participantCount: 678,
                    hasNew: true
                }
            ],

            // ===== Tab 4: 宠物问答 =====
            qaRefreshing: false,

            qaCategory: 'all',

            qaCategories: [
                {
                    id: 'all',
                    name: '全部'
                },
                {
                    id: 'training',
                    name: '训练'
                },
                {
                    id: 'health',
                    name: '健康'
                },
                {
                    id: 'diet',
                    name: '饮食'
                },
                {
                    id: 'behavior',
                    name: '行为'
                }
            ],

            questions: [
                {
                    id: 'qa_001',
                    title: '狗狗总是咬沙发怎么办？',
                    expertAnswer: '这是分离焦虑的表现，建议增加运动量，提供咀嚼玩具，可以考虑使用信息素喷雾来缓解焦虑。',
                    expert: {
                        name: '王兽医',
                        avatar: 'https://images.unsplash.com/photo-1612349317150-e413f6a5b16d?w=100&h=100&fit=crop'
                    },
                    viewCount: 1200,
                    answerCount: 3,
                    isFollowing: false
                },
                {
                    id: 'qa_002',
                    title: '猫咪掉毛严重是什么原因？',
                    expertAnswer: '季节性换毛是正常现象，但如果掉毛过多可能是营养不良或皮肤病，建议定期梳毛并检查皮肤状况。',
                    expert: {
                        name: '李医生',
                        avatar: 'https://images.unsplash.com/photo-1594824476967-48c8b964273f?w=100&h=100&fit=crop'
                    },
                    viewCount: 890,
                    answerCount: 5,
                    isFollowing: true
                },
                {
                    id: 'qa_003',
                    title: '新手养狗需要注意什么？',
                    expertAnswer: null,
                    viewCount: 2300,
                    answerCount: 12,
                    isFollowing: false
                },
                {
                    id: 'qa_004',
                    title: '猫咪不吃新换的猫粮怎么办？',
                    expertAnswer: '换粮需要循序渐进，建议采用7日换粮法，新旧粮混合逐步过渡，避免突然更换造成肠胃不适。',
                    expert: {
                        name: '张营养师',
                        avatar: 'https://images.unsplash.com/photo-1622253692010-333f2da6031d?w=100&h=100&fit=crop'
                    },
                    viewCount: 567,
                    answerCount: 2,
                    isFollowing: false
                }
            ],

            // ===== Tab 5: 线下活动 =====
            activityRefreshing: false,

            activityType: 'all',

            activityTypes: [
                {
                    id: 'all',
                    name: '全部'
                },
                {
                    id: 'gathering',
                    name: '聚会'
                },
                {
                    id: 'competition',
                    name: '比赛'
                },
                {
                    id: 'clinic',
                    name: '义诊'
                },
                {
                    id: 'adoption',
                    name: '领养日'
                }
            ],

            activities: [
                {
                    id: 'activity_001',
                    title: '🎉 周末狗狗聚会',
                    coverImage: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=600&h=400&fit=crop',
                    location: {
                        name: '朝阳公园'
                    },
                    distance: 2000,
                    date: '3月20日',
                    time: '14:00',
                    registeredCount: 12,
                    maxCount: 30,
                    isRegistered: false
                },
                {
                    id: 'activity_002',
                    title: '🏆 萌宠才艺大赛',
                    coverImage: 'https://images.unsplash.com/photo-1587300003388-59208cc962cb?w=600&h=400&fit=crop',
                    location: {
                        name: '奥林匹克公园'
                    },
                    distance: 3500,
                    date: '3月25日',
                    time: '10:00',
                    registeredCount: 45,
                    maxCount: 100,
                    isRegistered: true
                },
                {
                    id: 'activity_003',
                    title: '🏥 免费宠物义诊',
                    coverImage: 'https://images.unsplash.com/photo-1628009368231-7603352989c3?w=600&h=400&fit=crop',
                    location: {
                        name: '宠物医院'
                    },
                    distance: 1500,
                    date: '3月22日',
                    time: '09:00',
                    registeredCount: 28,
                    maxCount: 50,
                    isRegistered: false
                }
            ],

            // ===== Tab 6: 好物推荐 =====
            productRefreshing: false,

            productCategory: 'all',

            productCategories: [
                {
                    id: 'all',
                    name: '全部'
                },
                {
                    id: 'food',
                    name: '主粮'
                },
                {
                    id: 'snack',
                    name: '零食'
                },
                {
                    id: 'toy',
                    name: '玩具'
                },
                {
                    id: 'supplies',
                    name: '用品'
                },
                {
                    id: 'medical',
                    name: '医疗'
                }
            ],

            products: [
                {
                    id: 'product_001',
                    name: '进口天然猫粮 无谷物配方',
                    image: 'https://images.unsplash.com/photo-1589924691995-400dc9ecc119?w=400&h=400&fit=crop',
                    price: 128,
                    likes: 234
                },
                {
                    id: 'product_002',
                    name: '宠物自动饮水机 循环过滤',
                    image: 'https://images.unsplash.com/photo-1581888227599-779811939961?w=400&h=400&fit=crop',
                    price: 89,
                    likes: 89
                },
                {
                    id: 'product_003',
                    name: '耐咬狗狗玩具球 发声款',
                    image: 'https://images.unsplash.com/photo-1560807707-8cc77767d783?w=400&h=400&fit=crop',
                    price: 29,
                    likes: 156
                },
                {
                    id: 'product_004',
                    name: '宠物洗护套装 温和配方',
                    image: 'https://images.unsplash.com/photo-1516734212186-a967f81ad0d7?w=400&h=400&fit=crop',
                    price: 68,
                    likes: 78
                }
            ],

            cartCount: 3,
            cartTotal: 256,
            isRegisteredKey: false,
            countKey: ''
        };
    }, // ===== 生命周期 =====
    onLoad() {
        // 保存原始用户数据并添加距离文本
        const allUsers = this.addDistanceTextFun(this.nearbyUsers);
        this.setData({
            allNearbyUsers: allUsers,
            nearbyUsers: allUsers,
            filteredNearbyUsers: allUsers,
            activities: this.addDistanceTextFun(this.activities)
        });
        this.loadFeeds();
        this.loadTopics();
        this.loadQuestions();
        this.loadActivities();
        this.loadProducts();
    },
    onShow() {
        // 刷新购物车数据
        this.refreshCart();
    },
    methods: {
        // ===== 辅助方法 =====
        formatDistanceFun(meters) {
            if (meters < 1000) {
                return meters + 'm';
            }
            return (meters / 1000).toFixed(1) + 'km';
        },

        addDistanceTextFun(list) {
            return list.map((item) => ({
                ...item,
                distanceText: this.formatDistanceFun(item.distance)
            }));
        },

        // ===== Tab切换 =====
        onTabTap(e) {
            const index = parseInt(e.currentTarget.dataset.index);
            this.setData({
                currentTab: index
            });
        },

        onSwiperChange(e) {
            const index = e.detail.current;
            this.setData({
                currentTab: index
            });
        },

        // ===== 顶部搜索和消息 =====
        onSearchTap() {
            uni.showToast({
                title: '搜索功能开发中',
                icon: 'none'
            });
        },

        onMessageTap() {
            uni.navigateTo({
                url: '/pages/message/message'
            });
        },

        // ===== Tab 1: 附近宠友 =====
        onChangeLocation() {
            uni.chooseLocation({
                success: (res) => {
                    this.setData({
                        'location.name': res.name || res.address,
                        'location.latitude': res.latitude,
                        'location.longitude': res.longitude
                    });
                    this.loadNearbyUsers();
                }
            });
        },

        onNearbyFilterTap(e) {
            const filter = e.currentTarget.dataset.filter;
            this.setData({
                nearbyFilter: filter
            });
            this.filterNearbyUsers(filter);
        },

        // 根据距离筛选附近宠友
        filterNearbyUsers(filter) {
            const allUsers = this.allNearbyUsers;
            let filtered = [];
            switch (filter) {
                case '1km':
                    filtered = allUsers.filter((user) => user.distance <= 1000);
                    break;
                case '3km':
                    filtered = allUsers.filter((user) => user.distance <= 3000);
                    break;
                case '5km':
                    filtered = allUsers.filter((user) => user.distance <= 5000);
                    break;
                case 'all':
                default:
                    filtered = allUsers;
                    break;
            }

            // 按距离排序（由近到远）
            filtered.sort((a, b) => a.distance - b.distance);
            this.setData({
                nearbyUsers: filtered,
                filteredNearbyUsers: filtered
            });

            // 显示筛选结果提示
            const filterText = filter === 'all' ? '全部' : filter;
            uni.showToast({
                title: `显示${filterText}范围内宠友`,
                icon: 'none',
                duration: 1500
            });
        },

        onNearbyRefresh() {
            this.setData({
                nearbyRefreshing: true
            });
            setTimeout(() => {
                this.loadNearbyUsers();
                this.setData({
                    nearbyRefreshing: false
                });
            }, 1000);
        },

        loadNearbyUsers() {
            // 模拟加载附近用户
            console.log('加载附近用户，筛选:', this.nearbyFilter);
        },

        loadMoreNearby() {
            uni.showToast({
                title: '加载更多',
                icon: 'none'
            });
        },

        onUserCardTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/user-profile?id=${id}`
            });
        },

        onGreetTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.showToast({
                title: '已发送打招呼',
                icon: 'success'
            });
        },

        onViewProfileTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/user-profile?id=${id}`
            });
        },

        // ===== Tab 2: 动态广场 =====
        onFeedRefresh() {
            this.setData({
                feedRefreshing: true
            });
            setTimeout(() => {
                this.loadFeeds();
                this.setData({
                    feedRefreshing: false
                });
            }, 1000);
        },

        loadFeeds() {
            // 模拟加载动态
            console.log('加载动态列表');
        },

        loadMoreFeed() {
            uni.showToast({
                title: '加载更多',
                icon: 'none'
            });
        },

        onFeedCardTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/feed-detail?id=${id}`
            });
        },

        // ===== Tab 3: 话题讨论 =====
        onTopicSearchTap() {
            uni.showToast({
                title: '搜索话题',
                icon: 'none'
            });
        },

        onTopicCategoryTap(e) {
            const id = e.currentTarget.dataset.id;
            this.setData({
                topicCategory: id
            });
            this.loadTopics();
        },

        onTopicRefresh() {
            this.setData({
                topicRefreshing: true
            });
            setTimeout(() => {
                this.loadTopics();
                this.setData({
                    topicRefreshing: false
                });
            }, 1000);
        },

        loadTopics() {
            console.log('加载话题列表，分类:', this.topicCategory);
        },

        onTopicTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/topic-detail?id=${id}`
            });
        },

        // ===== Tab 4: 宠物问答 =====
        onQaSearchTap() {
            uni.showToast({
                title: '搜索问题',
                icon: 'none'
            });
        },

        onAskQuestionTap() {
            uni.navigateTo({
                url: '/pages/discover/question-create'
            });
        },

        onQaCategoryTap(e) {
            const id = e.currentTarget.dataset.id;
            this.setData({
                qaCategory: id
            });
            this.loadQuestions();
        },

        onQaRefresh() {
            this.setData({
                qaRefreshing: true
            });
            setTimeout(() => {
                this.loadQuestions();
                this.setData({
                    qaRefreshing: false
                });
            }, 1000);
        },

        loadQuestions() {
            console.log('加载问答列表，分类:', this.qaCategory);
        },

        onQuestionTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/question-detail?id=${id}`
            });
        },

        onFollowQuestionTap(e) {
            const id = e.currentTarget.dataset.id;
            const index = this.questions.findIndex((q) => q.id === id);
            if (index !== -1) {
                const key = `questions[${index}].isFollowing`;
                this.setData({
                    [key]: !this.questions[index].isFollowing
                });
                uni.showToast({
                    title: this.questions[index].isFollowing ? '已取消关注' : '已关注',
                    icon: 'none'
                });
            }
        },

        // ===== Tab 5: 线下活动 =====
        onActivitySearchTap() {
            uni.showToast({
                title: '搜索活动',
                icon: 'none'
            });
        },

        onPublishActivityTap() {
            uni.navigateTo({
                url: '/pages/discover/activity-create'
            });
        },

        onActivityTypeTap(e) {
            const id = e.currentTarget.dataset.id;
            this.setData({
                activityType: id
            });
            this.loadActivities();
        },

        onActivityRefresh() {
            this.setData({
                activityRefreshing: true
            });
            setTimeout(() => {
                this.loadActivities();
                this.setData({
                    activityRefreshing: false
                });
            }, 1000);
        },

        loadActivities() {
            console.log('加载活动列表，类型:', this.activityType);
        },

        onActivityTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/activity-detail?id=${id}`
            });
        },

        onJoinActivityTap(e) {
            const id = e.currentTarget.dataset.id;
            const index = this.activities.findIndex((a) => a.id === id);
            if (index !== -1) {
                if (this.activities[index].isRegistered) {
                    uni.showToast({
                        title: '您已报名',
                        icon: 'none'
                    });
                    return;
                }
                const isRegisteredKey = `activities[${index}].isRegistered`;
                const countKey = `activities[${index}].registeredCount`;
                this.setData({
                    [isRegisteredKey]: true,
                    [countKey]: this.activities[index].registeredCount + 1
                });
                uni.showToast({
                    title: '报名成功',
                    icon: 'success'
                });
            }
        },

        // ===== Tab 6: 好物推荐 =====
        onProductSearchTap() {
            uni.showToast({
                title: '搜索商品',
                icon: 'none'
            });
        },

        onCartTap() {
            uni.navigateTo({
                url: '/pages/discover/cart'
            });
        },

        onProductCategoryTap(e) {
            const id = e.currentTarget.dataset.id;
            this.setData({
                productCategory: id
            });
            this.loadProducts();
        },

        onProductRefresh() {
            this.setData({
                productRefreshing: true
            });
            setTimeout(() => {
                this.loadProducts();
                this.setData({
                    productRefreshing: false
                });
            }, 1000);
        },

        loadProducts() {
            console.log('加载商品列表，分类:', this.productCategory);
        },

        onProductTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/product-detail?id=${id}`
            });
        },

        onAddCartTap(e) {
            const id = e.currentTarget.dataset.id;
            const index = parseInt(e.currentTarget.dataset.index);
            const product = this.products[index];

            // 更新购物车
            this.setData({
                cartCount: this.cartCount + 1,
                cartTotal: this.cartTotal + product.price
            });
            uni.showToast({
                title: '已加入购物车',
                icon: 'success'
            });
        },

        refreshCart() {
            // 从存储中刷新购物车数据
            console.log('刷新购物车数据');
        },

        onCheckoutTap() {
            uni.navigateTo({
                url: '/pages/discover/order-confirm'
            });
        },

        // ===== 底部发布按钮 =====
        onFabPublishTap() {
            const itemList = ['发布动态', '发起话题', '提问', '发布活动'];
            uni.showActionSheet({
                itemList,
                success: (res) => {
                    const urls = ['/pages/discover/post-create', '/pages/discover/topic-create', '/pages/discover/question-create', '/pages/discover/activity-create'];
                    uni.navigateTo({
                        url: urls[res.tapIndex]
                    });
                }
            });
        }
    }
};
</script>
<style>
@import './discover.css';
</style>
