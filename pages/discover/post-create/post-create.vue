<template>
    <view class="post-create-page">
        <!-- 内容输入区 -->
        <view class="content-section">
            <textarea class="content-input" placeholder="分享你和宠物的精彩瞬间..." maxlength="500" @input="onContentInput" :value="content"></textarea>
            <text class="char-count">{{ content.length }}/500</text>
        </view>

        <!-- 图片上传区 -->
        <view class="image-section">
            <view class="image-grid">
                <view class="image-item" :data-index="index" v-for="(item, index) in images" :key="index">
                    <image class="uploaded-image" :src="item" mode="aspectFill"></image>

                    <view class="delete-btn" :data-index="index" @tap="onDeleteImage">
                        <text class="delete-icon">×</text>
                    </view>
                </view>
                <view class="upload-btn" v-if="images.length < 9" @tap="onChooseImage">
                    <text class="upload-icon">+</text>
                    <text class="upload-text">{{ images.length }}/9</text>
                </view>
            </view>
            <text class="upload-tip">最多上传9张图片</text>
        </view>

        <!-- 位置选择 -->
        <view class="option-section" @tap="onChooseLocation">
            <view class="option-left">
                <text class="option-icon">📍</text>
                <text class="option-text">{{ location || '添加位置' }}</text>
            </view>
            <text class="option-arrow">></text>
        </view>

        <!-- 话题标签 -->
        <view class="option-section" @tap="onChooseTopic">
            <view class="option-left">
                <text class="option-icon">#</text>
                <text class="option-text">{{ selectedTopic || '添加话题' }}</text>
            </view>
            <text class="option-arrow">></text>
        </view>

        <!-- 推荐话题 -->
        <view class="suggested-topics" v-if="suggestedTopics.length > 0">
            <text class="section-title">推荐话题</text>
            <view class="topic-tags">
                <view :class="'topic-tag ' + (selectedTopic === item ? 'selected' : '')" :data-topic="item" @tap="onTopicTap" v-for="(item, index) in suggestedTopics" :key="index">
                    <text>#{{ item }}</text>
                </view>
            </view>
        </view>

        <!-- 底部发布栏 -->
        <view class="bottom-bar">
            <button class="cancel-btn" @tap="onCancel">取消</button>
            <button :class="'publish-btn ' + (canPublish ? 'active' : '')" @tap="onPublish">发布</button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            content: '',
            images: [],
            location: '',
            selectedTopic: '',
            suggestedTopics: ['我家宠物的奇怪习惯', '狗狗日常', '猫咪萌照', '养宠小知识', '萌宠搞笑时刻', '宠物美食'],
            canPublish: false
        };
    },
    onLoad() {
        // 页面加载
    },
    methods: {
        // 内容输入
        onContentInput(e) {
            const content = e.detail.value;
            this.setData({
                content: content,
                canPublish: content.trim().length > 0 || this.images.length > 0
            });
        },

        // 选择图片
        onChooseImage() {
            const remainingCount = 9 - this.images.length;
            uni.chooseMedia({
                count: remainingCount,
                mediaType: ['image'],
                sourceType: ['album', 'camera'],
                success: (res) => {
                    const newImages = res.tempFiles.map((file) => file.tempFilePath);
                    const images = [...this.images, ...newImages];
                    this.setData({
                        images: images.slice(0, 9),
                        canPublish: images.length > 0 || this.content.trim().length > 0
                    });
                }
            });
        },

        // 删除图片
        onDeleteImage(e) {
            const { index } = e.currentTarget.dataset;
            const images = this.images.filter((_, i) => i !== index);
            this.setData({
                images,
                canPublish: images.length > 0 || this.content.trim().length > 0
            });
        },

        // 选择位置
        onChooseLocation() {
            uni.chooseLocation({
                success: (res) => {
                    this.setData({
                        location: res.name
                    });
                }
            });
        },

        // 选择话题
        onChooseTopic() {
            uni.showActionSheet({
                itemList: this.suggestedTopics,
                success: (res) => {
                    this.setData({
                        selectedTopic: this.suggestedTopics[res.tapIndex]
                    });
                }
            });
        },

        // 点击推荐话题
        onTopicTap(e) {
            const { topic } = e.currentTarget.dataset;
            const newTopic = this.selectedTopic === topic ? '' : topic;
            this.setData({
                selectedTopic: newTopic
            });
        },

        // 取消发布
        onCancel() {
            if (this.content || this.images.length > 0) {
                uni.showModal({
                    title: '提示',
                    content: '确定要放弃发布吗？',
                    success: (res) => {
                        if (res.confirm) {
                            uni.navigateBack();
                        }
                    }
                });
            } else {
                uni.navigateBack();
            }
        },

        // 发布动态
        onPublish() {
            const { content, images, location, selectedTopic, canPublish } = this;
            if (!canPublish) {
                uni.showToast({
                    title: '请输入内容或上传图片',
                    icon: 'none'
                });
                return;
            }
            uni.showLoading({
                title: '发布中...'
            });

            // 模拟发布
            setTimeout(() => {
                uni.hideLoading();
                uni.showToast({
                    title: '发布成功',
                    icon: 'success',
                    success: () => {
                        setTimeout(() => {
                            uni.navigateBack();
                        }, 1500);
                    }
                });
            }, 1500);
        }
    }
};
</script>
<style>
@import './post-create.css';
</style>
