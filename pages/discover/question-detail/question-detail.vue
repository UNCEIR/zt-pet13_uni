<template>
    <view class="question-detail-page">
        <!-- 问题头部 -->
        <view class="question-header card">
            <view class="question-category-tag">{{ question.categoryName }}</view>
            <view class="question-title">{{ question.title }}</view>
            <view class="question-content">{{ question.content }}</view>

            <!-- 提问者信息 -->
            <view class="author-info">
                <image class="author-avatar" :src="question.author.avatar" mode="aspectFill"></image>
                <view class="author-meta">
                    <text class="author-name">{{ question.author.nickname }}</text>
                    <text class="question-time">{{ question.createdAt }}</text>
                </view>
            </view>

            <!-- 问题统计 -->
            <view class="question-stats">
                <view class="stat-item">
                    <text class="stat-icon">👁</text>
                    <text class="stat-value">{{ question.stats.views }}</text>
                </view>
                <view class="stat-item">
                    <text class="stat-icon">💬</text>
                    <text class="stat-value">{{ question.stats.answerCount }}个回答</text>
                </view>
                <view class="stat-item">
                    <text class="stat-icon">⭐</text>
                    <text class="stat-value">{{ question.stats.follows }}关注</text>
                </view>
            </view>

            <!-- 关注问题按钮 -->
            <button :class="'follow-question-btn ' + (question.isFollowing ? 'following' : '')" @tap="onFollowTap">
                {{ question.isFollowing ? '已关注' : '关注问题' }}
            </button>
        </view>

        <!-- 回答列表 -->
        <view class="answers-section">
            <view class="section-header">
                <text class="section-title">全部回答 ({{ answers.length }})</text>
                <view class="sort-options">
                    <text :class="'sort-item ' + (sortBy === 'default' ? 'active' : '')" data-sort="default" @tap="onSortTap">默认</text>
                    <text :class="'sort-item ' + (sortBy === 'time' ? 'active' : '')" data-sort="time" @tap="onSortTap">最新</text>
                </view>
            </view>

            <!-- 回答列表 -->
            <view class="answer-list">
                <view class="answer-card card" v-for="(item, index) in answers" :key="index">
                    <!-- 回答者信息 -->

                    <view class="answer-header">
                        <image class="answerer-avatar" :src="item.author.avatar" mode="aspectFill"></image>
                        <view class="answerer-info">
                            <view class="answerer-name-row">
                                <text class="answerer-name">{{ item.author.nickname }}</text>
                                <view class="expert-badge" v-if="item.author.isExpert">
                                    <text class="badge-icon">✓</text>
                                    <text class="badge-text">{{ item.author.expertTitle }}</text>
                                </view>
                            </view>
                            <text class="answer-time">{{ item.createdAt }}</text>
                        </view>
                        <view class="accepted-mark" v-if="item.isAccepted">✓ 采纳</view>
                    </view>

                    <!-- 回答内容 -->

                    <view class="answer-content">{{ item.content }}</view>

                    <!-- 回答图片 -->

                    <view class="answer-images" v-if="item.images && item.images.length > 0">
                        <image
                            class="answer-image"
                            :src="img"
                            mode="aspectFill"
                            @tap="onPreviewImage"
                            :data-urls="item.images"
                            :data-current="img"
                            v-for="(img, index1) in item.images"
                            :key="index1"
                        ></image>
                    </view>

                    <!-- 回答操作 -->

                    <view class="answer-actions">
                        <view :class="'action-btn ' + (item.isLiked ? 'active' : '')" :data-id="item.id" @tap="onLikeTap">
                            <text class="action-icon">❤</text>
                            <text class="action-count">{{ item.likes }}</text>
                        </view>
                        <view class="action-btn" :data-id="item.id" @tap="onCommentTap">
                            <text class="action-icon">💬</text>
                            <text class="action-count">{{ item.comments }}</text>
                        </view>
                        <view class="action-btn" @tap="onShareTap">
                            <text class="action-icon">↗</text>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 加载更多 -->
            <view class="load-more" v-if="hasMore" @tap="loadMoreAnswers">
                <text>加载更多</text>
            </view>
        </view>

        <!-- 推荐相关问题 -->
        <view class="related-section" v-if="relatedQuestions.length > 0">
            <view class="section-header">
                <text class="section-title">相关问题</text>
            </view>
            <view class="related-list">
                <view class="related-item" :data-id="item.id" @tap="onRelatedQuestionTap" v-for="(item, index) in relatedQuestions" :key="index">
                    <text class="related-title">Q: {{ item.title }}</text>

                    <text class="related-count">{{ item.answerCount }}个回答</text>
                </view>
            </view>
        </view>

        <!-- 底部操作栏 -->
        <view class="bottom-bar">
            <view class="invite-btn" @tap="onInviteAnswerTap">
                <text class="invite-icon">@</text>
                <text class="invite-text">邀请回答</text>
            </view>
            <button class="answer-btn" @tap="onAnswerTap">写回答</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            questionId: '',

            question: {
                id: '',
                title: '',
                content: '',
                category: '',
                categoryName: '',
                author: {
                    id: '',
                    nickname: '',
                    avatar: ''
                },
                stats: {
                    views: 0,
                    answerCount: 0,
                    follows: 0
                },
                isFollowing: false,
                createdAt: ''
            },

            answers: [],
            sortBy: 'default',
            hasMore: true,
            page: 1,
            pageSize: 10,
            relatedQuestions: [],
            img: ''
        };
    },
    onLoad(options) {
        const { id } = options;
        if (!id) {
            uni.showToast({
                title: '问题ID不存在',
                icon: 'none'
            });
            uni.navigateBack();
            return;
        }
        this.setData({
            questionId: id
        });
        this.loadQuestionDetail(id);
        this.loadAnswers();
        this.loadRelatedQuestions();
    },
    onPullDownRefresh() {
        this.setData({
            page: 1,
            hasMore: true
        });
        Promise.all([this.loadQuestionDetail(this.questionId), this.loadAnswers(true)]).then(() => {
            uni.stopPullDownRefresh();
        });
    },
    onReachBottom() {
        if (this.hasMore) {
            this.loadMoreAnswers();
        }
    },
    // 分享给朋友
    onShareAppMessage() {
        return {
            title: `Q: ${this.question.title}`,
            path: `/pages/discover/question-detail?id=${this.questionId}`,
            imageUrl: '/images/share/question.png'
        };
    },
    // 分享到朋友圈
    onShareTimeline() {
        return {
            title: `Q: ${this.question.title}`,
            query: `id=${this.questionId}`,
            imageUrl: '/images/share/question.png'
        };
    },
    methods: {
        // 加载问题详情
        loadQuestionDetail(id) {
            // 模拟数据
            const question = {
                id: id,
                title: '狗狗总是咬沙发怎么办？',
                content:
                    '我家金毛今年2岁了，最近几个月总是咬沙发，特别是我们不在家的时候。已经试过很多方法，比如喷苦味剂、给玩具，但都没什么效果。有没有什么好的办法可以纠正这个行为？',
                category: 'behavior',
                categoryName: '行为',
                author: {
                    id: 'user_001',
                    nickname: '金毛主人',
                    avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                },
                stats: {
                    views: 1200,
                    answerCount: 3,
                    follows: 23
                },
                isFollowing: false,
                createdAt: '3天前'
            };
            this.setData({
                question
            });

            // 实际开发时使用:
            // wx.request({
            //   url: `${app.globalData.baseUrl}/questions/${id}`,
            //   success: (res) => {
            //     this.setData({ question: res.data });
            //   }
            // });
        },

        // 加载回答列表
        loadAnswers(refresh = false) {
            const page = refresh ? 1 : this.page;

            // 模拟数据
            const answers = [
                {
                    id: 'answer_001',
                    author: {
                        id: 'expert_001',
                        nickname: '王兽医',
                        avatar: 'https://images.unsplash.com/photo-1612349317150-e413f6a5b16d?w=100&h=100&fit=crop',
                        isExpert: true,
                        expertTitle: '认证兽医'
                    },
                    content:
                        '这是典型的分离焦虑表现。建议从以下几个方面入手：\n\n1. 增加运动量：金毛需要大量运动，每天早晚各1小时的散步或跑步可以消耗多余精力。\n\n2. 提供咀嚼玩具：选择耐咬的玩具，如KONG玩具，里面可以塞一些零食增加吸引力。\n\n3. 脱敏训练：假装出门但不离开，逐步延长离开时间，让狗狗适应独处。\n\n4. 考虑使用信息素喷雾或扩散器，可以缓解焦虑情绪。\n\n如果以上方法无效，建议咨询专业训犬师。',
                    images: [],
                    likes: 45,
                    comments: 8,
                    isLiked: false,
                    isAccepted: true,
                    createdAt: '2天前'
                },
                {
                    id: 'answer_002',
                    author: {
                        id: 'user_002',
                        nickname: '训犬达人',
                        avatar: 'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100&h=100&fit=crop',
                        isExpert: false,
                        expertTitle: ''
                    },
                    content:
                        '我家也有类似经历，分享我的经验：\n\n首先需要找出根本原因，可能是无聊、焦虑或者只是习惯性行为。\n\n可以尝试"替代行为训练"：当发现狗狗要咬沙发时，立即用玩具吸引注意力，一旦它开始玩玩具就给予奖励。长期坚持会有效果。',
                    images: ['https://images.unsplash.com/photo-1587300003388-59208cc962cb?w=400&h=300&fit=crop'],
                    likes: 28,
                    comments: 5,
                    isLiked: true,
                    isAccepted: false,
                    createdAt: '1天前'
                },
                {
                    id: 'answer_003',
                    author: {
                        id: 'user_003',
                        nickname: '宠物爱好者',
                        avatar: 'https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=100&h=100&fit=crop',
                        isExpert: false,
                        expertTitle: ''
                    },
                    content: '可以试着给狗狗穿上衣服，有些狗狗穿上衣服后会变得更安静，减少破坏行为。不过这个方法对部分狗狗有效，可以尝试一下。',
                    images: [],
                    likes: 12,
                    comments: 3,
                    isLiked: false,
                    isAccepted: false,
                    createdAt: '12小时前'
                }
            ];
            this.setData({
                answers: refresh ? answers : [...this.answers, ...answers],
                hasMore: answers.length >= this.pageSize
            });
        },

        // 加载更多回答
        loadMoreAnswers() {
            if (!this.hasMore) {
                return;
            }
            this.setData({
                page: this.page + 1
            });
            this.loadAnswers();
        },

        // 加载相关问题
        loadRelatedQuestions() {
            // 模拟数据
            const relatedQuestions = [
                {
                    id: 'qa_002',
                    title: '猫咪抓沙发怎么办？',
                    answerCount: 5
                },
                {
                    id: 'qa_003',
                    title: '如何纠正狗狗拆家行为？',
                    answerCount: 8
                },
                {
                    id: 'qa_004',
                    title: '金毛幼犬需要多少运动量？',
                    answerCount: 12
                }
            ];
            this.setData({
                relatedQuestions
            });
        },

        // 关注问题
        onFollowTap() {
            const isFollowing = !this.question.isFollowing;
            const follows = isFollowing ? this.question.stats.follows + 1 : this.question.stats.follows - 1;
            this.setData({
                'question.isFollowing': isFollowing,
                'question.stats.follows': follows
            });
            uni.showToast({
                title: isFollowing ? '已关注问题' : '已取消关注',
                icon: 'none'
            });
        },

        // 排序切换
        onSortTap(e) {
            const sortBy = e.currentTarget.dataset.sort;
            this.setData({
                sortBy,
                page: 1
            });
            this.loadAnswers(true);
        },

        // 点赞回答
        onLikeTap(e) {
            const id = e.currentTarget.dataset.id;
            const index = this.answers.findIndex((a) => a.id === id);
            if (index === -1) {
                return;
            }
            const isLiked = !this.answers[index].isLiked;
            const likes = isLiked ? this.answers[index].likes + 1 : this.answers[index].likes - 1;
            this.setData({
                [`answers[${index}].isLiked`]: isLiked,
                [`answers[${index}].likes`]: likes
            });
            uni.showToast({
                title: isLiked ? '点赞成功' : '取消点赞',
                icon: 'none'
            });
        },

        // 评论回答
        onCommentTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/answer-comments?answerId=${id}`
            });
        },

        // 分享
        onShareTap() {
            uni.showShareMenu({
                withShareTicket: true,
                menus: ['shareAppMessage', 'shareTimeline']
            });
        },

        // 预览图片
        onPreviewImage(e) {
            const { urls, current } = e.currentTarget.dataset;
            uni.previewImage({
                urls,
                current
            });
        },

        // 点击相关问题
        onRelatedQuestionTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/question-detail?id=${id}`
            });
        },

        // 邀请回答
        onInviteAnswerTap() {
            uni.showActionSheet({
                itemList: ['邀请好友', '邀请专家', '分享到群'],
                success: (res) => {
                    console.log('邀请方式:', res.tapIndex);
                }
            });
        },

        // 写回答
        onAnswerTap() {
            uni.navigateTo({
                url: `/pages/discover/answer-create?questionId=${this.questionId}`
            });
        }
    }
};
</script>
<style>
@import './question-detail.css';
</style>
