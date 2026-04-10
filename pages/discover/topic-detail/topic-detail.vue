<template>
    <view class="topic-detail-page">
        <!-- 话题信息头部 -->
        <view class="topic-header">
            <view class="topic-title-section">
                <text class="topic-tag">#{{ topic.name }}</text>
                <text class="topic-heat">🔥 {{ topic.heat }}</text>
            </view>
            <text class="topic-desc">{{ topic.description }}</text>
            <view class="topic-stats">
                <view class="stat-item">
                    <text class="stat-num">{{ topic.participantCount }}</text>
                    <text class="stat-label">人参与</text>
                </view>
                <view class="stat-item">
                    <text class="stat-num">{{ topic.postCount }}</text>
                    <text class="stat-label">个帖子</text>
                </view>
            </view>
            <button :class="'join-btn ' + (topic.isJoined ? 'joined' : '')" @tap="onJoinTap">
                {{ topic.isJoined ? '已参与' : '参与话题' }}
            </button>
        </view>

        <!-- 帖子列表 -->
        <view class="post-list-section">
            <view class="sort-bar">
                <text class="sort-label">排序:</text>
                <view class="sort-options">
                    <text :class="'sort-option ' + (sort === 'hot' ? 'active' : '')" data-sort="hot" @tap="onSortChange">最热</text>
                    <text :class="'sort-option ' + (sort === 'new' ? 'active' : '')" data-sort="new" @tap="onSortChange">最新</text>
                </view>
            </view>

            <scroll-view :scroll-y="true" class="post-scroll" :refresher-enabled="true" :refresher-triggered="refreshing" @refresherrefresh="onRefresh">
                <view class="post-list">
                    <view class="post-item" :data-id="item.id" @tap="onPostTap" v-for="(item, index) in posts" :key="index">
                        <view class="post-author">
                            <image class="author-avatar" :src="item.author.avatar" mode="aspectFill"></image>
                            <view class="author-info">
                                <text class="author-name">{{ item.author.name }}</text>
                                <text class="post-time">{{ item.time }}</text>
                            </view>
                        </view>

                        <text class="post-content">{{ item.content }}</text>

                        <view class="post-images" v-if="item.images.length > 0">
                            <image
                                :class="'post-image ' + (item.images.length === 1 ? 'single' : '')"
                                :src="img"
                                mode="aspectFill"
                                v-for="(img, index1) in item.images"
                                :key="index1"
                            ></image>
                        </view>

                        <view class="post-actions">
                            <view class="action-btn" @tap="onLikeTap" :data-id="item.id">
                                <text :class="'action-icon ' + (item.isLiked ? 'liked' : '')">❤️</text>
                                <text :class="'action-count ' + (item.isLiked ? 'liked' : '')">{{ item.likes }}</text>
                            </view>
                            <view class="action-btn" @tap="onCommentTap" :data-id="item.id">
                                <text class="action-icon">💬</text>
                                <text class="action-count">{{ item.comments }}</text>
                            </view>
                            <view class="action-btn" @tap="onShareTap" :data-id="item.id">
                                <text class="action-icon">↗️</text>
                                <text class="action-count">分享</text>
                            </view>
                        </view>
                    </view>
                </view>

                <view class="load-more" v-if="hasMore" @tap="loadMore">
                    <text>加载更多</text>
                </view>
            </scroll-view>
        </view>

        <!-- 底部参与栏 -->
        <view class="bottom-bar">
            <input class="comment-input" placeholder="参与话题讨论..." @tap="onInputTap" />
            <button class="join-topic-btn" @tap="onJoinTopic">参与话题</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            topicId: '',

            topic: {
                id: 'topic_001',
                name: '我家宠物的奇怪习惯',
                description: '分享你家宠物的各种奇葩习惯，看看谁家的最搞笑',
                heat: '2.3k',
                participantCount: 1200,
                postCount: 356,
                isJoined: false
            },

            sort: 'hot',
            refreshing: false,
            hasMore: true,

            posts: [
                {
                    id: 'post_001',
                    author: {
                        name: '豆豆麻麻',
                        avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                    },
                    content: '我家金毛豆豆每次都要抱着我的拖鞋睡觉，不给就一直叫！这是什么奇怪的癖好啊哈哈',
                    images: ['https://images.unsplash.com/photo-1552053831-71594a27632d?w=400&h=400&fit=crop'],
                    time: '2小时前',
                    likes: 128,
                    comments: 45,
                    isLiked: true
                },
                {
                    id: 'post_002',
                    author: {
                        name: '咪咪主人',
                        avatar: 'https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=100&h=100&fit=crop'
                    },
                    content: '我家布偶猫每次喝水都要先用爪子碰一下水面，好像在测试水深一样，超级可爱！',
                    images: [
                        'https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=400&h=400&fit=crop',
                        'https://images.unsplash.com/photo-1573865526739-10659fec78a5?w=400&h=400&fit=crop'
                    ],
                    time: '4小时前',
                    likes: 89,
                    comments: 23,
                    isLiked: false
                },
                {
                    id: 'post_003',
                    author: {
                        name: '旺财粑粑',
                        avatar: 'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=100&h=100&fit=crop'
                    },
                    content: '柴犬有个怪癖：每次出门回来都要在门槛上蹭半天才肯进屋，好像要把外面的味道擦掉',
                    images: [],
                    time: '6小时前',
                    likes: 56,
                    comments: 18,
                    isLiked: false
                }
            ],

            likesKey: '',
            img: ''
        };
    },
    onLoad(options) {
        const { id } = options;
        if (id) {
            this.setData({
                topicId: id
            });
            this.loadTopicDetail(id);
        }
    },
    // 分享
    onShareAppMessage() {
        const { topic } = this;
        return {
            title: `#${topic.name} - 来参与讨论吧！`,
            path: `/pages/discover/topic-detail?id=${topic.id}`
        };
    },
    methods: {
        // 加载话题详情
        loadTopicDetail(topicId) {
            console.log('加载话题详情:', topicId);
            // 这里应该调用API获取真实数据
        },

        // 参与/取消参与话题
        onJoinTap() {
            const { topic } = this;
            this.setData({
                'topic.isJoined': !topic.isJoined,
                'topic.participantCount': topic.isJoined ? topic.participantCount - 1 : topic.participantCount + 1
            });
            uni.showToast({
                title: topic.isJoined ? '已取消参与' : '已参与话题',
                icon: 'none'
            });
        },

        // 切换排序
        onSortChange(e) {
            const { sort } = e.currentTarget.dataset;
            this.setData({
                sort
            });
            this.loadPosts(sort);
        },

        // 加载帖子
        loadPosts(sort) {
            console.log('加载帖子列表,排序:', sort);
            // 这里应该调用API获取真实数据
        },

        // 下拉刷新
        onRefresh() {
            this.setData({
                refreshing: true
            });
            setTimeout(() => {
                this.loadPosts(this.sort);
                this.setData({
                    refreshing: false
                });
            }, 1000);
        },

        // 加载更多
        loadMore() {
            uni.showToast({
                title: '加载更多帖子',
                icon: 'none'
            });
        },

        // 点击帖子
        onPostTap(e) {
            const { id } = e.currentTarget.dataset;
            uni.navigateTo({
                url: `/pages/discover/feed-detail?id=${id}`
            });
        },

        // 点赞
        onLikeTap(e) {
            e.stopPropagation();
            const { id } = e.currentTarget.dataset;
            const index = this.posts.findIndex((p) => p.id === id);
            if (index !== -1) {
                const key = `posts[${index}].isLiked`;
                const likesKey = `posts[${index}].likes`;
                const isLiked = this.posts[index].isLiked;
                this.setData({
                    [key]: !isLiked,
                    [likesKey]: isLiked ? this.posts[index].likes - 1 : this.posts[index].likes + 1
                });
            }
        },

        // 评论
        onCommentTap(e) {
            e.stopPropagation();
            const { id } = e.currentTarget.dataset;
            uni.navigateTo({
                url: `/pages/discover/feed-detail?id=${id}`
            });
        },

        // 分享
        onShareTap(e) {
            e.stopPropagation();
            uni.showShareMenu({
                withShareTicket: true,
                menus: ['shareAppMessage', 'shareTimeline']
            });
        },

        // 输入框点击
        onInputTap() {
            // 跳转到发布页面，预填充话题
            uni.navigateTo({
                url: `/pages/discover/post-create?topic=${this.topic.name}`
            });
        },

        // 参与话题按钮
        onJoinTopic() {
            uni.navigateTo({
                url: `/pages/discover/post-create?topic=${this.topic.name}`
            });
        }
    }
};
</script>
<style>
@import './topic-detail.css';
</style>
