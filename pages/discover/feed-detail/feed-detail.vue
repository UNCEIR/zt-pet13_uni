<template>
    <view class="feed-detail-page">
        <!-- 作者信息 -->
        <view class="author-section">
            <image class="author-avatar" :src="feed.author.avatar" mode="aspectFill"></image>
            <view class="author-info">
                <text class="author-name">{{ feed.author.name }}</text>
                <text class="publish-time">{{ feed.publishTime }}</text>
            </view>
            <button :class="'follow-btn ' + (feed.isFollowing ? 'following' : '')" @tap="onFollowTap">
                {{ feed.isFollowing ? '已关注' : '+ 关注' }}
            </button>
        </view>

        <!-- 动态内容 -->
        <view class="content-section">
            <text class="feed-content">{{ feed.content }}</text>
            <view :class="'image-grid ' + (feed.images.length === 1 ? 'single' : feed.images.length === 2 ? 'two' : 'multi')">
                <image
                    class="feed-image"
                    :src="item"
                    mode="aspectFill"
                    :data-index="index"
                    @tap="onImagePreview"
                    :lazy-load="true"
                    v-for="(item, index) in feed.images"
                    :key="index"
                ></image>
            </view>
        </view>

        <!-- 位置信息 -->
        <view class="location-section" v-if="feed.location">
            <text class="location-icon">📍</text>
            <text class="location-text">{{ feed.location }}</text>
        </view>

        <!-- 互动数据 -->
        <view class="stats-section">
            <view class="stat-item" @tap="onLikeTap">
                <text :class="'stat-icon ' + (feed.isLiked ? 'liked' : '')">❤️</text>
                <text :class="'stat-count ' + (feed.isLiked ? 'liked' : '')">{{ feed.likes }}</text>
            </view>
            <view class="stat-item">
                <text class="stat-icon">💬</text>
                <text class="stat-count">{{ feed.comments }}</text>
            </view>
            <view class="stat-item" @tap="onShareTap">
                <text class="stat-icon">↗️</text>
                <text class="stat-count">分享</text>
            </view>
            <view class="stat-item" @tap="onCollectTap">
                <text :class="'stat-icon ' + (feed.isCollected ? 'collected' : '')">⭐</text>
                <text :class="'stat-count ' + (feed.isCollected ? 'collected' : '')">{{ feed.isCollected ? '已收藏' : '收藏' }}</text>
            </view>
        </view>

        <!-- 评论列表 -->
        <view class="comments-section">
            <view class="section-header">
                <text class="section-title">评论 ({{ commentList.length }})</text>
                <view class="sort-tabs">
                    <text :class="'sort-tab ' + (commentSort === 'hot' ? 'active' : '')" data-sort="hot" @tap="onSortChange">最热</text>
                    <text :class="'sort-tab ' + (commentSort === 'new' ? 'active' : '')" data-sort="new" @tap="onSortChange">最新</text>
                </view>
            </view>

            <view class="comment-list">
                <view class="comment-item" v-for="(item, index) in commentList" :key="index">
                    <image class="comment-avatar" :src="item.author.avatar" mode="aspectFill"></image>

                    <view class="comment-body">
                        <view class="comment-header">
                            <text class="comment-name">{{ item.author.name }}</text>
                            <text class="comment-time">{{ item.time }}</text>
                        </view>
                        <text class="comment-content">{{ item.content }}</text>
                        <view class="comment-actions">
                            <text class="reply-btn" :data-id="item.id" @tap="onReplyTap">回复</text>
                            <view class="like-btn" :data-id="item.id" @tap="onCommentLikeTap">
                                <text :class="'like-icon ' + (item.isLiked ? 'liked' : '')">❤️</text>
                                <text :class="'like-count ' + (item.isLiked ? 'liked' : '')">{{ item.likes }}</text>
                            </view>
                        </view>
                        <!-- 回复列表 -->
                        <view class="reply-list" v-if="item.replies && item.replies.length > 0">
                            <view class="reply-item" v-for="(reply, index1) in item.replies" :key="index1">
                                <text class="reply-author">{{ reply.author.name }}</text>

                                <text class="reply-to" v-if="reply.to">回复 {{ reply.to }}</text>

                                <text class="reply-content">：{{ reply.content }}</text>
                            </view>
                            <text class="more-replies" v-if="item.replyCount > 2" :data-id="item.id" @tap="onViewMoreReplies">展开 {{ item.replyCount - 2 }} 条回复</text>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 加载更多 -->
            <view class="load-more" v-if="hasMoreComments" @tap="loadMoreComments">
                <text>加载更多评论</text>
            </view>
        </view>

        <!-- 底部评论栏 -->
        <view class="comment-bar">
            <input
                class="comment-input"
                :placeholder="replyTo ? '回复 ' + replyTo + ':' : '写评论...'"
                :value="commentText"
                @input="onCommentInput"
                :focus="isInputFocus"
                @blur="onInputBlur"
            />
            <button :class="'send-btn ' + (commentText ? 'active' : '')" @tap="onSendComment">发送</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            feedId: '',

            feed: {
                id: 'feed_001',
                content: '今天带豆豆去公园玩，超级开心！天气真好，狗狗们都很活泼~ 🐕 #狗狗日常 #金毛 #公园',
                images: [
                    'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=600&h=600&fit=crop',
                    'https://images.unsplash.com/photo-1552053831-71594a27632d?w=600&h=800&fit=crop',
                    'https://images.unsplash.com/photo-1587300003388-59208cc962cb?w=600&h=600&fit=crop'
                ],
                location: '朝阳公园',
                author: {
                    name: '豆豆麻麻',
                    avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                },
                publishTime: '2小时前',
                likes: 128,
                comments: 23,
                isLiked: false,
                isFollowing: false,
                isCollected: false
            },

            commentSort: 'hot',
            commentText: '',
            replyTo: '',
            isInputFocus: false,
            hasMoreComments: true,

            commentList: [
                {
                    id: 'comment_001',
                    author: {
                        name: '奥利奥爸爸',
                        avatar: 'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100&h=100&fit=crop'
                    },
                    content: '豆豆好可爱！金毛真的太温顺了~',
                    time: '1小时前',
                    likes: 15,
                    isLiked: true,
                    replyCount: 2,
                    replies: [
                        {
                            id: 'reply_001',
                            author: {
                                name: '豆豆麻麻'
                            },
                            content: '谢谢夸奖，奥利奥也很帅气！'
                        },
                        {
                            id: 'reply_002',
                            author: {
                                name: '咪咪主人'
                            },
                            content: '羡慕有公园的，我们只能在家里玩'
                        }
                    ]
                },
                {
                    id: 'comment_002',
                    author: {
                        name: '旺财粑粑',
                        avatar: 'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=100&h=100&fit=crop'
                    },
                    content: '朝阳公园我经常去，下次可以约一起遛狗呀',
                    time: '45分钟前',
                    likes: 8,
                    isLiked: false,
                    replyCount: 1,
                    replies: [
                        {
                            id: 'reply_003',
                            author: {
                                name: '豆豆麻麻'
                            },
                            to: '旺财粑粑',
                            content: '好啊，周末一般都有空！'
                        }
                    ]
                },
                {
                    id: 'comment_003',
                    author: {
                        name: '花花麻麻',
                        avatar: 'https://images.unsplash.com/photo-1544005313-94ddf0286df2?w=100&h=100&fit=crop'
                    },
                    content: '猫咪表示也想出门玩，但是太胆小了哈哈',
                    time: '30分钟前',
                    likes: 5,
                    isLiked: false,
                    replyCount: 0,
                    replies: []
                }
            ],

            likesKey: '',

            reply: {
                author: {
                    name: ''
                },

                to: '',
                content: ''
            }
        };
    },
    onLoad(options) {
        const { id } = options;
        if (id) {
            this.setData({
                feedId: id
            });
            this.loadFeedDetail(id);
        }
    },
    // 分享功能
    onShareAppMessage() {
        const { feed } = this;
        return {
            title: feed.content.substring(0, 30) + '...',
            path: `/pages/discover/feed-detail?id=${feed.id}`,
            imageUrl: feed.images[0]
        };
    },
    methods: {
        // 加载动态详情
        loadFeedDetail(feedId) {
            console.log('加载动态详情:', feedId);
            // 这里应该调用API获取真实数据
        },

        // 关注/取消关注
        onFollowTap() {
            const { feed } = this;
            this.setData({
                'feed.isFollowing': !feed.isFollowing
            });
            uni.showToast({
                title: feed.isFollowing ? '已关注' : '已取消关注',
                icon: 'none'
            });
        },

        // 图片预览
        onImagePreview(e) {
            const { index } = e.currentTarget.dataset;
            const { images } = this.feed;
            uni.previewImage({
                current: images[index],
                urls: images
            });
        },

        // 点赞
        onLikeTap() {
            const { feed } = this;
            const newLiked = !feed.isLiked;
            const newLikes = newLiked ? feed.likes + 1 : feed.likes - 1;
            this.setData({
                'feed.isLiked': newLiked,
                'feed.likes': newLikes
            });
            uni.showToast({
                title: newLiked ? '点赞成功' : '取消点赞',
                icon: 'none'
            });
        },

        // 分享
        onShareTap() {
            uni.showShareMenu({
                withShareTicket: true,
                menus: ['shareAppMessage', 'shareTimeline']
            });
        },

        // 收藏
        onCollectTap() {
            const { feed } = this;
            this.setData({
                'feed.isCollected': !feed.isCollected
            });
            uni.showToast({
                title: feed.isCollected ? '已收藏' : '已取消收藏',
                icon: 'none'
            });
        },

        // 切换评论排序
        onSortChange(e) {
            const { sort } = e.currentTarget.dataset;
            this.setData({
                commentSort: sort
            });
            this.loadComments(sort);
        },

        // 加载评论
        loadComments(sort) {
            console.log('加载评论列表,排序:', sort);
        },

        // 回复评论
        onReplyTap(e) {
            const { id } = e.currentTarget.dataset;
            const comment = this.commentList.find((c) => c.id === id);
            if (comment) {
                this.setData({
                    replyTo: comment.author.name,
                    isInputFocus: true
                });
            }
        },

        // 评论点赞
        onCommentLikeTap(e) {
            const { id } = e.currentTarget.dataset;
            const index = this.commentList.findIndex((c) => c.id === id);
            if (index !== -1) {
                const key = `commentList[${index}].isLiked`;
                const likesKey = `commentList[${index}].likes`;
                const isLiked = this.commentList[index].isLiked;
                this.setData({
                    [key]: !isLiked,
                    [likesKey]: isLiked ? this.commentList[index].likes - 1 : this.commentList[index].likes + 1
                });
            }
        },

        // 查看更多回复
        onViewMoreReplies(e) {
            const { id } = e.currentTarget.dataset;
            console.log('查看更多回复:', id);
        },

        // 加载更多评论
        loadMoreComments() {
            uni.showToast({
                title: '加载更多评论',
                icon: 'none'
            });
        },

        // 评论输入
        onCommentInput(e) {
            this.setData({
                commentText: e.detail.value
            });
        },

        // 输入框失焦
        onInputBlur() {
            if (!this.commentText) {
                this.setData({
                    replyTo: '',
                    isInputFocus: false
                });
            }
        },

        // 发送评论
        onSendComment() {
            const { commentText, replyTo } = this;
            if (!commentText.trim()) {
                uni.showToast({
                    title: '请输入评论内容',
                    icon: 'none'
                });
                return;
            }
            console.log('发送评论:', commentText, '回复给:', replyTo);
            uni.showToast({
                title: '评论成功',
                icon: 'success'
            });
            this.setData({
                commentText: '',
                replyTo: '',
                isInputFocus: false
            });
        }
    }
};
</script>
<style>
@import './feed-detail.css';
</style>
