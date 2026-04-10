<template>
    <view class="user-profile-page">
        <!-- 顶部导航栏 -->
        <view class="nav-bar">
            <view class="nav-back" @tap="onBackTap">
                <text class="back-icon">←</text>
            </view>
            <text class="nav-title">用户主页</text>
            <view class="nav-more" @tap="onMoreTap">
                <text class="more-icon">⋯</text>
            </view>
        </view>

        <!-- 用户信息头部 -->
        <view class="profile-header">
            <!-- 封面图 -->
            <image class="cover-image" :src="user.coverImage" mode="aspectFill"></image>
            <view class="cover-overlay"></view>

            <!-- 用户信息区 -->
            <view class="user-info-section">
                <image class="user-avatar" :src="user.avatar" mode="aspectFill"></image>
                <view class="user-meta">
                    <text class="user-name">{{ user.nickname }}</text>
                    <view class="user-location">
                        <text class="location-icon">📍</text>
                        <text>{{ user.location }}</text>
                        <text class="dot-separator">·</text>
                        <text>养宠{{ user.petYears }}年</text>
                    </view>
                    <text class="user-bio" v-if="user.bio">{{ user.bio }}</text>
                </view>
            </view>

            <!-- 操作按钮 -->
            <view class="action-buttons">
                <button :class="'btn-follow ' + (user.isFollowing ? 'following' : '')" @tap="onFollowTap">
                    <text class="btn-icon" v-if="!user.isFollowing">+</text>
                    <text>{{ user.isFollowing ? '已关注' : '关注' }}</text>
                </button>
                <button class="btn-message" @tap="onMessageTap">
                    <text class="btn-icon">✉</text>
                    <text>私信</text>
                </button>
            </view>
        </view>

        <!-- 统计数据 -->
        <view class="stats-section">
            <view class="stat-item" @tap="onStatTap" data-type="following">
                <text class="stat-value">{{ user.stats.following }}</text>
                <text class="stat-label">关注</text>
            </view>
            <view class="stat-item" @tap="onStatTap" data-type="followers">
                <text class="stat-value">{{ user.stats.followers }}</text>
                <text class="stat-label">粉丝</text>
            </view>
            <view class="stat-item" @tap="onStatTap" data-type="likes">
                <text class="stat-value">{{ user.stats.likes }}</text>
                <text class="stat-label">获赞</text>
            </view>
        </view>

        <!-- 宠物卡片 -->
        <view class="pets-section" v-if="user.pets && user.pets.length > 0">
            <view class="section-header">
                <text class="section-title">TA的宠物</text>
                <text class="section-count">{{ user.pets.length }}只</text>
            </view>
            <scroll-view class="pets-scroll" :scroll-x="true" :show-scrollbar="false">
                <view class="pets-list">
                    <view class="pet-card" :data-id="item.id" @tap="onPetTap" v-for="(item, index) in user.pets" :key="index">
                        <image class="pet-avatar" :src="item.avatar" mode="aspectFill"></image>

                        <view class="pet-icon">{{ item.type === 'dog' ? '🐕' : item.type === 'cat' ? '🐱' : '🐾' }}</view>

                        <text class="pet-name">{{ item.name }}</text>

                        <text class="pet-breed">{{ item.breed }}</text>
                    </view>
                </view>
            </scroll-view>
        </view>

        <!-- 内容Tab -->
        <view class="content-tabs">
            <view :class="'tab-item ' + (currentTab === index ? 'active' : '')" :data-index="index" @tap="onTabTap" v-for="(item, index) in tabs" :key="index">
                <text class="tab-text">{{ item.name }}</text>

                <text class="tab-count" v-if="item.count > 0">{{ item.count }}</text>

                <view class="tab-indicator" v-if="currentTab === index"></view>
            </view>
        </view>

        <!-- Tab内容区域 -->
        <swiper class="content-swiper" :current="currentTab" @change="onSwiperChange" duration="200">
            <!-- Tab 1: 动态 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-scroll" :refresher-enabled="true" :refresher-triggered="feedRefreshing" @refresherrefresh="onFeedRefresh">
                    <view class="feed-list" v-if="feeds.length > 0">
                        <view class="feed-card" :data-id="item.id" @tap="onFeedTap" v-for="(item, index) in feeds" :key="index">
                            <view class="feed-image-wrap" v-if="item.images && item.images.length > 0">
                                <image class="feed-image" :src="item.images[0]" mode="aspectFill" :lazy-load="true"></image>
                                <view class="image-count" v-if="item.images.length > 1">+{{ item.images.length - 1 }}</view>
                            </view>

                            <text class="feed-content text-ellipsis-2">{{ item.content }}</text>

                            <view class="feed-stats">
                                <text class="stat">❤️ {{ item.likes }}</text>
                                <text class="stat">💬 {{ item.comments }}</text>
                            </view>
                        </view>
                    </view>
                    <view class="empty-state" v-else>
                        <text class="empty-icon">📝</text>
                        <text class="empty-text">还没有发布动态</text>
                    </view>
                </scroll-view>
            </swiper-item>

            <!-- Tab 2: 话题 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-scroll" :refresher-enabled="true" :refresher-triggered="topicRefreshing" @refresherrefresh="onTopicRefresh">
                    <view class="topic-list" v-if="topics.length > 0">
                        <view class="topic-item" :data-id="item.id" @tap="onTopicTap" v-for="(item, index) in topics" :key="index">
                            <view class="topic-tag">#{{ item.name }}</view>

                            <view class="topic-meta">
                                <text>{{ item.participantCount }}人参与</text>
                                <text class="topic-heat">🔥 {{ item.heat }}</text>
                            </view>
                        </view>
                    </view>
                    <view class="empty-state" v-else>
                        <text class="empty-icon">#</text>
                        <text class="empty-text">还没有参与话题</text>
                    </view>
                </scroll-view>
            </swiper-item>

            <!-- Tab 3: 问答 -->
            <swiper-item class="swiper-item">
                <scroll-view :scroll-y="true" class="tab-scroll" :refresher-enabled="true" :refresher-triggered="qaRefreshing" @refresherrefresh="onQaRefresh">
                    <view class="qa-list" v-if="questions.length > 0">
                        <view class="qa-item" :data-id="item.id" @tap="onQuestionTap" v-for="(item, index) in questions" :key="index">
                            <view class="qa-header">
                                <text class="qa-type">{{ item.type === 'ask' ? '提问' : '回答' }}</text>
                                <text class="qa-time">{{ item.time }}</text>
                            </view>

                            <text class="qa-title text-ellipsis">{{ item.title }}</text>

                            <view class="qa-meta">
                                <text class="qa-views">👁️ {{ item.viewCount }}</text>
                                <text class="qa-answers" v-if="item.answerCount > 0">{{ item.answerCount }}个回答</text>
                            </view>
                        </view>
                    </view>
                    <view class="empty-state" v-else>
                        <text class="empty-icon">❓</text>
                        <text class="empty-text">还没有问答记录</text>
                    </view>
                </scroll-view>
            </swiper-item>
        </swiper>
    </view>
</template>

<script>
export default {
    data() {
        return {
            // 当前用户ID
            userId: '',
            // 用户信息
            user: {
                id: 'user_001',
                nickname: '豆豆麻麻',
                avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=200&h=200&fit=crop',
                coverImage: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=750&h=400&fit=crop',
                location: '北京',
                petYears: 3,
                bio: '爱狗人士 | 金毛豆豆的主人 | 喜欢分享宠物日常',
                isFollowing: false,
                stats: {
                    following: 128,
                    followers: 256,
                    likes: 1024
                },
                pets: [
                    {
                        id: 'pet_001',
                        name: '豆豆',
                        type: 'dog',
                        breed: '金毛寻回犬',
                        avatar: 'https://images.unsplash.com/photo-1552053831-71594a27632d?w=200&h=200&fit=crop'
                    }
                ]
            },
            // Tab配置
            tabs: [
                {
                    name: '动态',
                    count: 12
                },
                {
                    name: '话题',
                    count: 5
                },
                {
                    name: '问答',
                    count: 3
                }
            ],
            currentTab: 0,
            // 刷新状态
            feedRefreshing: false,
            topicRefreshing: false,
            qaRefreshing: false,
            // 动态数据
            feeds: [
                {
                    id: 'feed_001',
                    content: '今天带豆豆去公园玩，超级开心！🐕 豆豆跑得飞快，还认识了很多新朋友～',
                    images: [
                        'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=400&h=500&fit=crop',
                        'https://images.unsplash.com/photo-1552053831-71594a27632d?w=400&h=400&fit=crop'
                    ],
                    likes: 128,
                    comments: 23
                },
                {
                    id: 'feed_002',
                    content: '给豆豆新买的玩具，它超级喜欢！',
                    images: ['https://images.unsplash.com/photo-1576201836106-db17578fd166?w=400&h=400&fit=crop'],
                    likes: 56,
                    comments: 8
                },
                {
                    id: 'feed_003',
                    content: '周末带豆豆去爬山，景色超美！虽然有点累，但是看到豆豆开心的样子，一切都值得了～',
                    images: [
                        'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?w=400&h=350&fit=crop',
                        'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=400&h=400&fit=crop',
                        'https://images.unsplash.com/photo-1552053831-71594a27632d?w=400&h=400&fit=crop'
                    ],
                    likes: 89,
                    comments: 15
                }
            ],
            // 话题数据
            topics: [
                {
                    id: 'topic_001',
                    name: '我家宠物的奇怪习惯',
                    participantCount: 1200,
                    heat: '2.3k'
                },
                {
                    id: 'topic_003',
                    name: '狗狗训练心得交流',
                    participantCount: 2100,
                    heat: '3.1k'
                },
                {
                    id: 'topic_005',
                    name: '领养代替购买',
                    participantCount: 678,
                    heat: '1.2k'
                }
            ],
            // 问答数据
            questions: [
                {
                    id: 'qa_001',
                    type: 'ask',
                    title: '狗狗总是咬沙发怎么办？',
                    time: '2天前',
                    viewCount: 1200,
                    answerCount: 3
                },
                {
                    id: 'qa_003',
                    type: 'ask',
                    title: '新手养狗需要注意什么？',
                    time: '1周前',
                    viewCount: 2300,
                    answerCount: 12
                },
                {
                    id: 'qa_005',
                    type: 'answer',
                    title: '金毛犬的饮食建议',
                    time: '3天前',
                    viewCount: 567,
                    answerCount: 0
                }
            ]
        };
    }, // ===== 生命周期 =====
    onLoad(options) {
        const userId = options.id || options.userId || '';
        this.setData({
            userId
        });
        if (userId) {
            this.loadUserData(userId);
        }
    },
    onShow() {
        // 刷新数据
        this.refreshCurrentTab();
    },
    onShareAppMessage() {
        const { user } = this;
        return {
            title: `${user.nickname}的个人主页`,
            path: `/pages/discover/user-profile?id=${user.id}`,
            imageUrl: user.avatar
        };
    },
    onShareTimeline() {
        const { user } = this;
        return {
            title: `${user.nickname}的个人主页 - 养宠${user.petYears}年`,
            query: `id=${user.id}`,
            imageUrl: user.coverImage
        };
    },
    methods: {
        // ===== 数据加载 =====
        loadUserData(userId) {
            // 模拟加载用户数据
            console.log('加载用户数据:', userId);

            // 实际项目中这里应该调用接口
            // wx.request({
            //   url: `/api/users/${userId}`,
            //   success: (res) => {
            //     this.setData({ user: res.data });
            //   }
            // });
        },

        refreshCurrentTab() {
            const { currentTab } = this;
            switch (currentTab) {
                case 0:
                    this.loadFeeds();
                    break;
                case 1:
                    this.loadTopics();
                    break;
                case 2:
                    this.loadQuestions();
                    break;
            }
        },

        // ===== 导航操作 =====
        onBackTap() {
            uni.navigateBack();
        },

        onMoreTap() {
            uni.showActionSheet({
                itemList: ['分享主页', '举报用户', '加入黑名单'],
                success: (res) => {
                    switch (res.tapIndex) {
                        case 0:
                            this.shareProfile();
                            break;
                        case 1:
                            uni.showToast({
                                title: '举报功能开发中',
                                icon: 'none'
                            });
                            break;
                        case 2:
                            uni.showModal({
                                title: '确认加入黑名单',
                                content: '加入黑名单后将不再看到该用户的内容',
                                success: (modalRes) => {
                                    if (modalRes.confirm) {
                                        uni.showToast({
                                            title: '已加入黑名单',
                                            icon: 'success'
                                        });
                                    }
                                }
                            });
                            break;
                    }
                }
            });
        },

        // ===== 关注/私信操作 =====
        onFollowTap() {
            const { user } = this;
            const newFollowingState = !user.isFollowing;
            this.setData({
                'user.isFollowing': newFollowingState,
                'user.stats.followers': newFollowingState ? user.stats.followers + 1 : user.stats.followers - 1
            });
            uni.showToast({
                title: newFollowingState ? '关注成功' : '已取消关注',
                icon: 'success'
            });

            // 实际项目中这里应该调用接口
            // wx.request({
            //   url: `/api/users/${user.id}/follow`,
            //   method: newFollowingState ? 'POST' : 'DELETE'
            // });
        },

        onMessageTap() {
            const { user } = this;
            uni.navigateTo({
                url: `/pages/message/chat?userId=${user.id}&nickname=${user.nickname}`
            });
        },

        // ===== 统计数据点击 =====
        onStatTap(e) {
            const type = e.currentTarget.dataset.type;
            const { user } = this;
            switch (type) {
                case 'following':
                    uni.navigateTo({
                        url: `/pages/discover/user-list?type=following&userId=${user.id}`
                    });
                    break;
                case 'followers':
                    uni.navigateTo({
                        url: `/pages/discover/user-list?type=followers&userId=${user.id}`
                    });
                    break;
                case 'likes':
                    uni.showToast({
                        title: `共获得 ${user.stats.likes} 个赞`,
                        icon: 'none'
                    });
                    break;
            }
        },

        // ===== 宠物卡片 =====
        onPetTap(e) {
            const petId = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/pet-detail/pet-detail?id=${petId}`
            });
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

        // ===== 动态Tab =====
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
            console.log('加载用户动态');
            // 实际项目中调用接口
        },

        onFeedTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/feed-detail?id=${id}`
            });
        },

        // ===== 话题Tab =====
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
            console.log('加载用户话题');
            // 实际项目中调用接口
        },

        onTopicTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/topic-detail?id=${id}`
            });
        },

        // ===== 问答Tab =====
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
            console.log('加载用户问答');
            // 实际项目中调用接口
        },

        onQuestionTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/question-detail?id=${id}`
            });
        },

        // ===== 分享 =====
        shareProfile() {
            uni.showShareMenu({
                withShareTicket: true,
                menus: ['shareAppMessage', 'shareTimeline']
            });
        }
    }
};
</script>
<style>
@import './user-profile.css';
</style>
