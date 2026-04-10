<template>
    <view class="activity-detail-page">
        <!-- 活动封面 -->
        <view class="cover-section">
            <image class="activity-cover" :src="activity.coverImage" mode="aspectFill"></image>
            <view class="cover-overlay">
                <view class="activity-type-tag">{{ activity.typeName }}</view>
                <view :class="'activity-status ' + activity.status">{{ activity.statusText }}</view>
            </view>
        </view>

        <!-- 活动信息卡片 -->
        <view class="info-card card">
            <view class="activity-title">{{ activity.title }}</view>

            <!-- 活动基本信息 -->
            <view class="info-list">
                <view class="info-item">
                    <view class="info-icon location">📍</view>
                    <view class="info-content">
                        <view class="info-label">活动地点</view>
                        <view class="info-value">{{ activity.location.name }}</view>
                        <view class="info-sub">{{ activity.location.address }}</view>
                    </view>
                </view>

                <view class="info-item">
                    <view class="info-icon time">📅</view>
                    <view class="info-content">
                        <view class="info-label">活动时间</view>
                        <view class="info-value">{{ activity.date }} {{ activity.time }}</view>
                        <view class="info-sub">{{ activity.duration }}</view>
                    </view>
                </view>

                <view class="info-item">
                    <view class="info-icon people">👥</view>
                    <view class="info-content">
                        <view class="info-label">报名人数</view>
                        <view class="info-value">{{ activity.capacity.current }}/{{ activity.capacity.max }}人</view>
                        <view class="progress-bar">
                            <view class="progress-fill" :style="'width: ' + (activity.capacity.current / activity.capacity.max) * 100 + '%'"></view>
                        </view>
                        <view class="info-sub" v-if="activity.capacity.current >= activity.capacity.max">已满员</view>
                        <view class="info-sub" v-else-if="activity.capacity.max - activity.capacity.current <= 5">
                            仅剩{{ activity.capacity.max - activity.capacity.current }}个名额
                        </view>
                    </view>
                </view>

                <view class="info-item" v-if="activity.fee > 0">
                    <view class="info-icon fee">💰</view>
                    <view class="info-content">
                        <view class="info-label">活动费用</view>
                        <view class="info-value price">¥{{ activity.fee }}/人</view>
                    </view>
                </view>
                <view class="info-item" v-else>
                    <view class="info-icon fee">💰</view>
                    <view class="info-content">
                        <view class="info-label">活动费用</view>
                        <view class="info-value free">免费</view>
                    </view>
                </view>
            </view>
        </view>

        <!-- 组织者信息 -->
        <view class="organizer-card card">
            <view class="card-title">活动组织者</view>
            <view class="organizer-info" @tap="onOrganizerTap">
                <image class="organizer-avatar" :src="activity.organizer.avatar" mode="aspectFill"></image>
                <view class="organizer-detail">
                    <view class="organizer-name-row">
                        <text class="organizer-name">{{ activity.organizer.nickname }}</text>
                        <view class="verified-badge" v-if="activity.organizer.isVerified">
                            <text class="badge-icon">✓</text>
                            <text class="badge-text">认证用户</text>
                        </view>
                    </view>
                    <view class="organizer-stats">发布{{ activity.organizer.activityCount }}个活动 · {{ activity.organizer.followerCount }}人关注</view>
                </view>
                <view :class="'follow-btn ' + (activity.organizer.isFollowing ? 'following' : '')" @tap.stop.prevent="onFollowOrganizerTap">
                    {{ activity.organizer.isFollowing ? '已关注' : '关注' }}
                </view>
            </view>
        </view>

        <!-- 活动详情 -->
        <view class="detail-card card">
            <view class="card-title">活动详情</view>
            <view class="detail-content">{{ activity.description }}</view>

            <!-- 活动须知 -->
            <view class="notice-section" v-if="activity.notices && activity.notices.length > 0">
                <view class="notice-title">活动须知</view>
                <view class="notice-list">
                    <view class="notice-item" v-for="(item, index) in activity.notices" :key="index">
                        <text class="notice-dot">•</text>

                        <text class="notice-text">{{ item }}</text>
                    </view>
                </view>
            </view>

            <!-- 携带物品 -->
            <view class="items-section" v-if="activity.items && activity.items.length > 0">
                <view class="items-title">建议携带</view>
                <view class="items-list">
                    <view class="item-tag" v-for="(item, index) in activity.items" :key="index">{{ item }}</view>
                </view>
            </view>
        </view>

        <!-- 活动相册 -->
        <view class="gallery-card card" v-if="activity.gallery && activity.gallery.length > 0">
            <view class="card-title">活动相册</view>
            <scroll-view class="gallery-scroll" :scroll-x="true" :show-scrollbar="false">
                <view class="gallery-list">
                    <image
                        class="gallery-image"
                        :src="item"
                        mode="aspectFill"
                        @tap="onPreviewGallery"
                        :data-urls="activity.gallery"
                        :data-current="item"
                        v-for="(item, index) in activity.gallery"
                        :key="index"
                    ></image>
                </view>
            </scroll-view>
        </view>

        <!-- 已报名用户 -->
        <view class="participants-card card">
            <view class="card-header">
                <view class="card-title">已报名 ({{ participants.length }})</view>
                <view class="view-all" @tap="onViewAllParticipants" v-if="participants.length > 6">查看全部 ></view>
            </view>
            <view class="participants-list">
                <view class="participant-item" v-if="index < 6" @tap="onParticipantTap" :data-id="item.id" v-for="(item, index) in participants" :key="index">
                    <image class="participant-avatar" :src="item.avatar"></image>

                    <text class="participant-name">{{ item.name }}</text>
                </view>
            </view>
        </view>

        <!-- 地图位置 -->
        <view class="map-card card">
            <view class="card-title">活动地点</view>
            <view class="map-container" @tap="onOpenLocation">
                <view class="map-placeholder">
                    <text class="map-icon">🗺</text>
                    <text class="map-text">{{ activity.location.name }}</text>
                    <text class="map-hint">点击查看详细位置</text>
                </view>
            </view>
        </view>

        <!-- 底部操作栏 -->
        <view class="bottom-bar">
            <view class="action-buttons">
                <view :class="'action-btn ' + (activity.isCollected ? 'active' : '')" @tap="onCollectTap">
                    <text class="action-icon">⭐</text>
                    <text class="action-label">收藏</text>
                </view>
                <view class="action-btn" @tap="onShareTap">
                    <text class="action-icon">↗</text>
                    <text class="action-label">分享</text>
                </view>
                <view class="action-btn" @tap="onContactOrganizerTap">
                    <text class="action-icon">💬</text>
                    <text class="action-label">咨询</text>
                </view>
            </view>

            <button
                :class="'join-btn ' + (activity.isRegistered ? 'registered' : '') + ' ' + (activity.status !== 'open' ? 'disabled' : '')"
                @tap="onJoinTap"
                :disabled="activity.status !== 'open'"
            >
                {{ activity.isRegistered ? '已报名' : activity.statusText }}
            </button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            activityId: '',
            activity: {
                id: '',
                title: '',
                type: '',
                typeName: '',
                status: '',
                statusText: '',
                coverImage: '',
                description: '',
                location: {
                    name: '',
                    address: '',
                    lat: 0,
                    lng: 0
                },
                date: '',
                time: '',
                duration: '',
                capacity: {
                    current: 0,
                    max: 0
                },
                fee: 0,
                organizer: {
                    id: '',
                    nickname: '',
                    avatar: '',
                    isVerified: false,
                    activityCount: 0,
                    followerCount: 0,
                    isFollowing: false
                },
                notices: [],
                items: [],
                gallery: [],
                isRegistered: false,
                isCollected: false
            },
            participants: []
        };
    },
    onLoad(options) {
        const { id } = options;
        if (!id) {
            uni.showToast({
                title: '活动ID不存在',
                icon: 'none'
            });
            uni.navigateBack();
            return;
        }
        this.setData({
            activityId: id
        });
        this.loadActivityDetail(id);
        this.loadParticipants(id);
    },
    onPullDownRefresh() {
        Promise.all([this.loadActivityDetail(this.activityId), this.loadParticipants(this.activityId)]).then(() => {
            uni.stopPullDownRefresh();
        });
    },
    // 分享给朋友
    onShareAppMessage() {
        const { activity } = this;
        return {
            title: activity.title,
            path: `/pages/discover/activity-detail?id=${activity.id}`,
            imageUrl: activity.coverImage
        };
    },
    // 分享到朋友圈
    onShareTimeline() {
        const { activity } = this;
        return {
            title: activity.title,
            query: `id=${activity.id}`,
            imageUrl: activity.coverImage
        };
    },
    methods: {
        // 加载活动详情
        loadActivityDetail(id) {
            // 模拟数据
            const typeNames = {
                gathering: '聚会',
                competition: '比赛',
                clinic: '义诊',
                adoption: '领养日'
            };
            const statusMap = {
                open: {
                    text: '立即报名',
                    class: 'open'
                },
                full: {
                    text: '已满员',
                    class: 'full'
                },
                ended: {
                    text: '已结束',
                    class: 'ended'
                },
                cancelled: {
                    text: '已取消',
                    class: 'cancelled'
                }
            };
            const activity = {
                id: id,
                title: '🎉 周末狗狗聚会',
                type: 'gathering',
                typeName: typeNames['gathering'],
                status: 'open',
                statusText: statusMap['open'].text,
                coverImage: 'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=800&h=500&fit=crop',
                description:
                    '欢迎大家带着自家的毛孩子一起来参加周末聚会！\n\n这是一个让狗狗们自由奔跑、交友的好机会。现场会准备：\n• 狗狗玩具和零食\n• 饮用水和小食\n• 拍照打卡区\n\n让我们度过一个愉快的周末下午！',
                location: {
                    name: '朝阳公园',
                    address: '北京市朝阳区朝阳公园南路1号',
                    lat: 39.943,
                    lng: 116.484
                },
                date: '3月20日',
                time: '14:00',
                duration: '预计3小时',
                capacity: {
                    current: 12,
                    max: 30
                },
                fee: 0,
                organizer: {
                    id: 'user_123',
                    nickname: '宠物达人小王',
                    avatar: 'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100&h=100&fit=crop',
                    isVerified: true,
                    activityCount: 15,
                    followerCount: 356,
                    isFollowing: false
                },
                notices: ['请确保您的狗狗已完成疫苗接种', '请携带牵引绳和拾便袋', '活动当天请提前15分钟到场', '如遇恶劣天气，活动将顺延'],
                items: ['牵引绳', '拾便袋', '水碗', '狗狗零食'],
                gallery: [
                    'https://images.unsplash.com/photo-1601758228041-f3b2795255f1?w=400&h=300&fit=crop',
                    'https://images.unsplash.com/photo-1548199973-03cce0bbc87b?w=400&h=300&fit=crop',
                    'https://images.unsplash.com/photo-1587300003388-59208cc962cb?w=400&h=300&fit=crop'
                ],
                isRegistered: false,
                isCollected: false
            };
            this.setData({
                activity
            });
        },

        // 加载报名用户
        loadParticipants(id) {
            // 模拟数据
            const participants = [
                {
                    id: 'p1',
                    name: '豆豆麻麻',
                    avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                },
                {
                    id: 'p2',
                    name: '咪咪主人',
                    avatar: 'https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=100&h=100&fit=crop'
                },
                {
                    id: 'p3',
                    name: '旺财粑粑',
                    avatar: 'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=100&h=100&fit=crop'
                },
                {
                    id: 'p4',
                    name: '奥利奥爸爸',
                    avatar: 'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100&h=100&fit=crop'
                },
                {
                    id: 'p5',
                    name: '花花麻麻',
                    avatar: 'https://images.unsplash.com/photo-1544005313-94ddf0286df2?w=100&h=100&fit=crop'
                },
                {
                    id: 'p6',
                    name: '小黄主人',
                    avatar: 'https://images.unsplash.com/photo-1500648767791-00dcc994a43e?w=100&h=100&fit=crop'
                },
                {
                    id: 'p7',
                    name: '大白麻麻',
                    avatar: 'https://images.unsplash.com/photo-1534528741775-53994a69daeb?w=100&h=100&fit=crop'
                },
                {
                    id: 'p8',
                    name: '小黑主人',
                    avatar: 'https://images.unsplash.com/photo-1506794778202-cad84cf45f1d?w=100&h=100&fit=crop'
                }
            ];
            this.setData({
                participants
            });
        },

        // 点击组织者
        onOrganizerTap() {
            const { organizer } = this.activity;
            uni.navigateTo({
                url: `/pages/discover/user-profile?id=${organizer.id}`
            });
        },

        // 关注/取消关注组织者
        onFollowOrganizerTap(e) {
            e.stopPropagation();
            const isFollowing = !this.activity.organizer.isFollowing;
            this.setData({
                'activity.organizer.isFollowing': isFollowing
            });
            uni.showToast({
                title: isFollowing ? '已关注' : '已取消关注',
                icon: 'none'
            });
        },

        // 预览相册图片
        onPreviewGallery(e) {
            const { urls, current } = e.currentTarget.dataset;
            uni.previewImage({
                urls,
                current
            });
        },

        // 点击报名用户
        onParticipantTap(e) {
            const id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/discover/user-profile?id=${id}`
            });
        },

        // 查看全部报名用户
        onViewAllParticipants() {
            uni.navigateTo({
                url: `/pages/discover/activity-participants?activityId=${this.activityId}`
            });
        },

        // 打开地图位置
        onOpenLocation() {
            const { location } = this.activity;
            uni.openLocation({
                latitude: location.lat,
                longitude: location.lng,
                name: location.name,
                address: location.address
            });
        },

        // 收藏/取消收藏
        onCollectTap() {
            const isCollected = !this.activity.isCollected;
            this.setData({
                'activity.isCollected': isCollected
            });
            uni.showToast({
                title: isCollected ? '已收藏' : '已取消收藏',
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

        // 联系组织者
        onContactOrganizerTap() {
            uni.showActionSheet({
                itemList: ['发送私信', '拨打电话'],
                success: (res) => {
                    if (res.tapIndex === 0) {
                        uni.navigateTo({
                            url: `/pages/message/chat?userId=${this.activity.organizer.id}`
                        });
                    } else {
                        uni.makePhoneCall({
                            phoneNumber: '13800138000'
                        });
                    }
                }
            });
        },

        // 报名/取消报名
        onJoinTap() {
            const { activity } = this;
            if (activity.isRegistered) {
                // 取消报名确认
                uni.showModal({
                    title: '取消报名',
                    content: '确定要取消报名吗？',
                    confirmColor: '#ff4757',
                    success: (res) => {
                        if (res.confirm) {
                            this.setData({
                                'activity.isRegistered': false,
                                'activity.capacity.current': activity.capacity.current - 1
                            });
                            uni.showToast({
                                title: '已取消报名',
                                icon: 'success'
                            });
                        }
                    }
                });
            } else {
                // 报名
                if (activity.capacity.current >= activity.capacity.max) {
                    uni.showToast({
                        title: '活动已满员',
                        icon: 'none'
                    });
                    return;
                }

                // 付费活动需要支付
                if (activity.fee > 0) {
                    uni.navigateTo({
                        url: `/pages/discover/activity-pay?activityId=${activity.id}`
                    });
                    return;
                }
                this.setData({
                    'activity.isRegistered': true,
                    'activity.capacity.current': activity.capacity.current + 1
                });
                uni.showToast({
                    title: '报名成功',
                    icon: 'success'
                });

                // 添加到我的活动
                this.addToMyActivities(activity);
            }
        },

        // 添加到我的活动
        addToMyActivities(activity) {
            // 实际开发时保存到本地存储或服务器
            console.log('添加到我的活动:', activity);
        }
    }
};
</script>
<style>
@import './activity-detail.css';
</style>
