<template>
    <view class="activity-create-page">
        <!-- 活动封面 -->
        <view class="form-section card cover-section">
            <view class="section-label">
                <text class="label-text">活动封面</text>
                <text class="label-required">*</text>
            </view>
            <view class="cover-upload" @tap="onUploadCover">
                <image class="cover-preview" :src="coverImage" mode="aspectFill" v-if="coverImage"></image>
                <view class="upload-placeholder" v-else>
                    <text class="upload-icon">📷</text>
                    <text class="upload-text">点击上传封面</text>
                    <text class="upload-hint">建议尺寸 800x500</text>
                </view>
            </view>
        </view>

        <!-- 活动标题 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">活动标题</text>
                <text class="label-required">*</text>
            </view>
            <input class="form-input" placeholder="请输入活动标题，吸引人眼球" placeholder-class="input-placeholder" maxlength="50" :value="title" @input="onTitleInput" />
            <view class="input-count">{{ title.length }}/50</view>
        </view>

        <!-- 活动类型 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">活动类型</text>
                <text class="label-required">*</text>
            </view>
            <view class="type-list">
                <view :class="'type-item ' + (selectedType === item.id ? 'active' : '')" :data-id="item.id" @tap="onTypeSelect" v-for="(item, index) in activityTypes" :key="index">
                    <text class="type-icon">{{ item.icon }}</text>

                    <text class="type-name">{{ item.name }}</text>
                </view>
            </view>
        </view>

        <!-- 活动时间 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">活动时间</text>
                <text class="label-required">*</text>
            </view>
            <view class="time-row">
                <picker mode="date" :value="date" :start="minDate" @change="onDateChange">
                    <view class="picker-item">
                        <text class="picker-label">日期</text>
                        <text :class="'picker-value ' + (date ? '' : 'placeholder')">{{ date || '请选择日期' }}</text>
                        <text class="picker-arrow">></text>
                    </view>
                </picker>
            </view>
            <view class="time-row">
                <picker mode="time" :value="time" @change="onTimeChange">
                    <view class="picker-item">
                        <text class="picker-label">时间</text>
                        <text :class="'picker-value ' + (time ? '' : 'placeholder')">{{ time || '请选择时间' }}</text>
                        <text class="picker-arrow"></text>
                    </view>
                </picker>
            </view>
            <view class="time-row">
                <picker mode="selector" :range="durationOptions" :value="durationIndex" @change="onDurationChange">
                    <view class="picker-item">
                        <text class="picker-label">预计时长</text>
                        <text class="picker-value">{{ durationOptions[durationIndex] }}</text>
                        <text class="picker-arrow"></text>
                    </view>
                </picker>
            </view>
        </view>

        <!-- 活动地点 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">活动地点</text>
                <text class="label-required">*</text>
            </view>
            <view class="location-row" @tap="onChooseLocation">
                <view class="location-info">
                    <text :class="'location-name ' + (location.name ? '' : 'placeholder')">{{ location.name || '点击选择活动地点' }}</text>
                    <text class="location-address" v-if="location.address">{{ location.address }}</text>
                </view>
                <text class="location-arrow"></text>
            </view>
        </view>

        <!-- 人数限制 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">人数限制</text>
                <text class="label-required">*</text>
            </view>
            <view class="capacity-row">
                <text class="capacity-label">最多可报名</text>
                <input class="capacity-input" type="number" :value="maxCapacity" @input="onCapacityInput" />
                <text class="capacity-unit">人</text>
            </view>
            <view class="capacity-hint">建议人数：聚会10-50人，比赛50-200人</view>
        </view>

        <!-- 活动费用 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">活动费用</text>
            </view>
            <view class="fee-options">
                <view :class="'fee-option ' + (feeType === 'free' ? 'active' : '')" data-type="free" @tap="onFeeTypeChange">
                    <text class="fee-label">免费</text>
                </view>
                <view :class="'fee-option ' + (feeType === 'paid' ? 'active' : '')" data-type="paid" @tap="onFeeTypeChange">
                    <text class="fee-label">收费</text>
                </view>
            </view>
            <view class="fee-input-row" v-if="feeType === 'paid'">
                <text class="fee-symbol">¥</text>
                <input class="fee-input" type="digit" placeholder="请输入费用" :value="fee" @input="onFeeInput" />
                <text class="fee-unit">/人</text>
            </view>
        </view>

        <!-- 活动描述 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">活动描述</text>
                <text class="label-required">*</text>
            </view>
            <textarea
                class="form-textarea"
                placeholder="请详细描述活动内容，包括：\n1. 活动流程安排\n2. 注意事项\n3. 需要准备的物品\n4. 联系方式"
                placeholder-class="textarea-placeholder"
                maxlength="1000"
                :value="description"
                @input="onDescriptionInput"
            ></textarea>
            <view class="input-count">{{ description.length }}/1000</view>
        </view>

        <!-- 活动须知 -->
        <view class="form-section card">
            <view class="section-header">
                <view class="section-label">
                    <text class="label-text">活动须知</text>
                </view>
                <view class="add-btn" @tap="onAddNotice">
                    <text class="add-icon">+</text>
                    <text>添加</text>
                </view>
            </view>
            <view class="notice-list">
                <view class="notice-input-item" v-for="(item, index) in notices" :key="index">
                    <input class="notice-input" placeholder="请输入注意事项" :value="item" :data-index="index" @input="onNoticeInput" />

                    <view class="delete-notice-btn" :data-index="index" @tap="onDeleteNotice">
                        <text>×</text>
                    </view>
                </view>
            </view>
        </view>

        <!-- 建议携带 -->
        <view class="form-section card">
            <view class="section-header">
                <view class="section-label">
                    <text class="label-text">建议携带</text>
                </view>
                <view class="add-btn" @tap="onAddItem">
                    <text class="add-icon">+</text>
                    <text>添加</text>
                </view>
            </view>
            <view class="items-input-list">
                <view class="item-input-tag" v-for="(item, index) in items" :key="index">
                    <input class="item-input" placeholder="物品名称" :value="item" :data-index="index" @input="onItemInput" />

                    <view class="delete-item-btn" :data-index="index" @tap="onDeleteItem">
                        <text>×</text>
                    </view>
                </view>
            </view>
        </view>

        <!-- 活动相册 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">活动相册</text>
                <text class="label-hint">（选填，最多9张）</text>
            </view>
            <view class="image-list">
                <view class="image-item" v-for="(item, index) in gallery" :key="index">
                    <image class="gallery-image" :src="item" mode="aspectFill"></image>

                    <view class="delete-btn" :data-index="index" @tap="onDeleteGalleryImage">
                        <text>×</text>
                    </view>
                </view>
                <view class="add-image-btn" v-if="gallery.length < 9" @tap="onAddGalleryImage">
                    <text class="add-icon">+</text>
                    <text class="add-text">添加图片</text>
                </view>
            </view>
        </view>

        <!-- 发布按钮 -->
        <view class="bottom-bar">
            <button :class="'submit-btn ' + (!canSubmit ? 'disabled' : '')" @tap="onSubmit" :disabled="!canSubmit || isSubmitting">
                {{ isSubmitting ? '发布中...' : '发布活动' }}
            </button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            // 表单数据
            coverImage: '',

            title: '',
            selectedType: '',
            date: '',
            time: '',
            durationIndex: 1,

            location: {
                name: '',
                address: '',
                lat: 0,
                lng: 0
            },

            maxCapacity: 30,
            feeType: 'free',
            fee: '',
            description: '',
            notices: [],
            items: [],
            gallery: [],

            // 配置选项
            activityTypes: [
                {
                    id: 'gathering',
                    name: '聚会',
                    icon: '🎉'
                },
                {
                    id: 'competition',
                    name: '比赛',
                    icon: '🏆'
                },
                {
                    id: 'clinic',
                    name: '义诊',
                    icon: '🏥'
                },
                {
                    id: 'adoption',
                    name: '领养日',
                    icon: '🐾'
                }
            ],

            durationOptions: ['1小时', '2小时', '3小时', '4小时', '半天', '全天'],
            minDate: '',

            // 状态
            isSubmitting: false,

            canSubmit: false,
            activities: []
        };
    },
    onLoad() {
        // 设置最小日期为今天
        const today = new Date();
        const year = today.getFullYear();
        const month = String(today.getMonth() + 1).padStart(2, '0');
        const day = String(today.getDate()).padStart(2, '0');
        this.setData({
            minDate: `${year}-${month}-${day}`,
            selectedType: this.activityTypes[0].id
        });
        this.checkCanSubmit();
    },
    methods: {
        // 上传封面
        onUploadCover() {
            uni.chooseMedia({
                count: 1,
                mediaType: ['image'],
                sourceType: ['album', 'camera'],
                success: (res) => {
                    this.setData({
                        coverImage: res.tempFiles[0].tempFilePath
                    });
                    this.checkCanSubmit();
                }
            });
        },

        // 标题输入
        onTitleInput(e) {
            this.setData({
                title: e.detail.value
            });
            this.checkCanSubmit();
        },

        // 选择活动类型
        onTypeSelect(e) {
            this.setData({
                selectedType: e.currentTarget.dataset.id
            });
        },

        // 日期选择
        onDateChange(e) {
            this.setData({
                date: e.detail.value
            });
            this.checkCanSubmit();
        },

        // 时间选择
        onTimeChange(e) {
            this.setData({
                time: e.detail.value
            });
            this.checkCanSubmit();
        },

        // 时长选择
        onDurationChange(e) {
            this.setData({
                durationIndex: parseInt(e.detail.value)
            });
        },

        // 选择地点
        onChooseLocation() {
            uni.chooseLocation({
                success: (res) => {
                    this.setData({
                        location: {
                            name: res.name || res.address,
                            address: res.address,
                            lat: res.latitude,
                            lng: res.longitude
                        }
                    });
                    this.checkCanSubmit();
                }
            });
        },

        // 人数输入
        onCapacityInput(e) {
            const value = parseInt(e.detail.value) || 0;
            this.setData({
                maxCapacity: value
            });
            this.checkCanSubmit();
        },

        // 费用类型切换
        onFeeTypeChange(e) {
            const type = e.currentTarget.dataset.type;
            this.setData({
                feeType: type,
                fee: type === 'free' ? '' : this.fee
            });
        },

        // 费用输入
        onFeeInput(e) {
            this.setData({
                fee: e.detail.value
            });
        },

        // 描述输入
        onDescriptionInput(e) {
            this.setData({
                description: e.detail.value
            });
            this.checkCanSubmit();
        },

        // 添加须知
        onAddNotice() {
            if (this.notices.length >= 5) {
                uni.showToast({
                    title: '最多添加5条须知',
                    icon: 'none'
                });
                return;
            }
            this.setData({
                notices: [...this.notices, '']
            });
        },

        // 须知输入
        onNoticeInput(e) {
            const index = e.currentTarget.dataset.index;
            const notices = [...this.notices];
            notices[index] = e.detail.value;
            this.setData({
                notices
            });
        },

        // 删除须知
        onDeleteNotice(e) {
            const index = e.currentTarget.dataset.index;
            const notices = this.notices.filter((_, i) => i !== index);
            this.setData({
                notices
            });
        },

        // 添加物品
        onAddItem() {
            if (this.items.length >= 8) {
                uni.showToast({
                    title: '最多添加8个物品',
                    icon: 'none'
                });
                return;
            }
            this.setData({
                items: [...this.items, '']
            });
        },

        // 物品输入
        onItemInput(e) {
            const index = e.currentTarget.dataset.index;
            const items = [...this.items];
            items[index] = e.detail.value;
            this.setData({
                items
            });
        },

        // 删除物品
        onDeleteItem(e) {
            const index = e.currentTarget.dataset.index;
            const items = this.items.filter((_, i) => i !== index);
            this.setData({
                items
            });
        },

        // 添加相册图片
        onAddGalleryImage() {
            const remaining = 9 - this.gallery.length;
            if (remaining <= 0) {
                return;
            }
            uni.chooseMedia({
                count: remaining,
                mediaType: ['image'],
                sourceType: ['album', 'camera'],
                success: (res) => {
                    const newImages = res.tempFiles.map((f) => f.tempFilePath);
                    this.setData({
                        gallery: [...this.gallery, ...newImages]
                    });
                }
            });
        },

        // 删除相册图片
        onDeleteGalleryImage(e) {
            const index = e.currentTarget.dataset.index;
            const gallery = this.gallery.filter((_, i) => i !== index);
            this.setData({
                gallery
            });
        },

        // 检查是否可以提交
        checkCanSubmit() {
            const { coverImage, title, date, time, location, description, maxCapacity } = this;
            const canSubmit = coverImage && title.trim().length >= 5 && date && time && location.name && description.trim().length >= 20 && maxCapacity > 0;
            this.setData({
                canSubmit
            });
        },

        // 提交表单
        onSubmit() {
            if (!this.canSubmit || this.isSubmitting) {
                return;
            }

            // 表单验证
            const { title, description, maxCapacity, feeType, fee } = this;
            if (title.trim().length < 5) {
                uni.showToast({
                    title: '标题至少需要5个字',
                    icon: 'none'
                });
                return;
            }
            if (description.trim().length < 20) {
                uni.showToast({
                    title: '描述至少需要20个字',
                    icon: 'none'
                });
                return;
            }
            if (maxCapacity <= 0) {
                uni.showToast({
                    title: '请输入有效的人数限制',
                    icon: 'none'
                });
                return;
            }
            if (feeType === 'paid' && (!fee || parseFloat(fee) <= 0)) {
                uni.showToast({
                    title: '请输入有效的费用金额',
                    icon: 'none'
                });
                return;
            }
            this.setData({
                isSubmitting: true
            });

            // 模拟提交
            setTimeout(() => {
                const typeNames = {
                    gathering: '聚会',
                    competition: '比赛',
                    clinic: '义诊',
                    adoption: '领养日'
                };
                const newActivity = {
                    id: `activity_${Date.now()}`,
                    title: this.title,
                    type: this.selectedType,
                    typeName: typeNames[this.selectedType],
                    status: 'open',
                    statusText: '立即报名',
                    coverImage: this.coverImage,
                    description: this.description,
                    location: this.location,
                    date: this.date,
                    time: this.time,
                    duration: this.durationOptions[this.durationIndex],
                    capacity: {
                        current: 0,
                        max: this.maxCapacity
                    },
                    fee: this.feeType === 'paid' ? parseFloat(this.fee) : 0,
                    organizer: {
                        id: 'user_current',
                        nickname: '当前用户',
                        avatar: 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop',
                        isVerified: true,
                        activityCount: 5,
                        followerCount: 128,
                        isFollowing: false
                    },
                    notices: this.notices.filter((n) => n.trim()),
                    items: this.items.filter((i) => i.trim()),
                    gallery: this.gallery,
                    isRegistered: true,
                    isCollected: false
                };
                uni.showToast({
                    title: '发布成功',
                    icon: 'success'
                });

                // 返回发现页并刷新活动列表
                setTimeout(() => {
                    const pages = getCurrentPages();
                    const discoverPage = pages.find((p) => p.route === 'pages/discover/discover');
                    if (discoverPage) {
                        discoverPage.setData({
                            activities: [newActivity, ...discoverPage.data.activities]
                        });
                    }
                    uni.navigateBack();
                }, 1500);
                this.setData({
                    isSubmitting: false
                });
            }, 2000);
        }
    }
};
</script>
<style>
@import './activity-create.css';
</style>
