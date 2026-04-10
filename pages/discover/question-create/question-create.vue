<template>
    <view class="question-create-page">
        <!-- 问题标题 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">问题标题</text>
                <text class="label-required">*</text>
            </view>
            <input
                class="title-input"
                placeholder="请输入问题标题，简明扼要地描述你的问题"
                placeholder-class="input-placeholder"
                maxlength="100"
                :value="title"
                @input="onTitleInput"
            />
            <view class="input-count">{{ title.length }}/100</view>
        </view>

        <!-- 问题分类 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">问题分类</text>
                <text class="label-required">*</text>
            </view>
            <view class="category-list">
                <view
                    :class="'category-item ' + (selectedCategory === item.id ? 'active' : '')"
                    :data-id="item.id"
                    @tap="onCategorySelect"
                    v-for="(item, index) in categories"
                    :key="index"
                >
                    <text class="category-icon">{{ item.icon }}</text>

                    <text class="category-name">{{ item.name }}</text>
                </view>
            </view>
        </view>

        <!-- 问题描述 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">问题描述</text>
                <text class="label-hint">详细描述有助于获得更好的回答</text>
            </view>
            <textarea
                class="content-textarea"
                placeholder="请详细描述你的问题，包括：\n1. 宠物的基本情况（品种、年龄、性别）\n2. 问题的具体表现\n3. 已经尝试过的解决方法\n4. 期望得到什么样的帮助"
                placeholder-class="textarea-placeholder"
                maxlength="1000"
                :value="content"
                @input="onContentInput"
            ></textarea>
            <view class="input-count">{{ content.length }}/1000</view>
        </view>

        <!-- 添加图片 -->
        <view class="form-section card">
            <view class="section-label">
                <text class="label-text">添加图片</text>
                <text class="label-hint">（选填，最多9张）</text>
            </view>
            <view class="image-list">
                <view class="image-item" v-for="(item, index) in images" :key="index">
                    <image class="uploaded-image" :src="item" mode="aspectFill"></image>

                    <view class="delete-btn" :data-index="index" @tap="onDeleteImage">
                        <text class="delete-icon">×</text>
                    </view>
                </view>
                <view class="add-image-btn" v-if="images.length < 9" @tap="onAddImage">
                    <text class="add-icon">+</text>
                    <text class="add-text">添加图片</text>
                </view>
            </view>
        </view>

        <!-- 匿名提问 -->
        <view class="form-section card anonymous-section">
            <view class="anonymous-row" @tap="onAnonymousToggle">
                <view class="anonymous-info">
                    <text class="anonymous-label">匿名提问</text>
                    <text class="anonymous-hint">开启后其他用户无法看到你的昵称和头像</text>
                </view>
                <switch class="anonymous-switch" :checked="isAnonymous" color="#56ab2f"></switch>
            </view>
        </view>

        <!-- 提问提示 -->
        <view class="tips-section">
            <view class="tips-title">
                <text class="tips-icon">💡</text>
                <text>提问小贴士</text>
            </view>
            <view class="tips-list">
                <view class="tip-item">• 标题简明扼要，直接描述核心问题</view>
                <view class="tip-item">• 描述中提供足够的背景信息</view>
                <view class="tip-item">• 选择正确的分类有助于获得专业回答</view>
                <view class="tip-item">• 上传相关图片可以帮助专家更好判断</view>
            </view>
        </view>

        <!-- 底部操作栏 -->
        <view class="bottom-bar">
            <button :class="'submit-btn ' + (!canSubmit ? 'disabled' : '')" @tap="onSubmit" :disabled="!canSubmit || isSubmitting">
                {{ isSubmitting ? '发布中...' : '发布问题' }}
            </button>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            title: '',
            content: '',
            selectedCategory: '',
            images: [],
            isAnonymous: false,
            isSubmitting: false,
            canSubmit: false,

            categories: [
                {
                    id: 'training',
                    name: '训练',
                    icon: '🎾'
                },
                {
                    id: 'health',
                    name: '健康',
                    icon: '🏥'
                },
                {
                    id: 'diet',
                    name: '饮食',
                    icon: '🍖'
                },
                {
                    id: 'behavior',
                    name: '行为',
                    icon: '🐾'
                }
            ],

            questions: []
        };
    },
    onLoad() {
        // 默认选中第一个分类
        this.setData({
            selectedCategory: this.categories[0].id
        });
        this.checkCanSubmit();
    },
    methods: {
        // 标题输入
        onTitleInput(e) {
            const title = e.detail.value.trim();
            this.setData({
                title
            });
            this.checkCanSubmit();
        },

        // 内容输入
        onContentInput(e) {
            const content = e.detail.value;
            this.setData({
                content
            });
            this.checkCanSubmit();
        },

        // 选择分类
        onCategorySelect(e) {
            const id = e.currentTarget.dataset.id;
            this.setData({
                selectedCategory: id
            });
        },

        // 添加图片
        onAddImage() {
            const remainingCount = 9 - this.images.length;
            if (remainingCount <= 0) {
                uni.showToast({
                    title: '最多添加9张图片',
                    icon: 'none'
                });
                return;
            }
            uni.chooseMedia({
                count: remainingCount,
                mediaType: ['image'],
                sourceType: ['album', 'camera'],
                success: (res) => {
                    const newImages = res.tempFiles.map((file) => file.tempFilePath);
                    this.setData({
                        images: [...this.images, ...newImages]
                    });
                }
            });
        },

        // 删除图片
        onDeleteImage(e) {
            const index = e.currentTarget.dataset.index;
            const images = this.images.filter((_, i) => i !== index);
            this.setData({
                images
            });
        },

        // 匿名开关
        onAnonymousToggle() {
            this.setData({
                isAnonymous: !this.isAnonymous
            });
        },

        // 检查是否可以提交
        checkCanSubmit() {
            const { title, content, selectedCategory } = this;
            const canSubmit = title.trim().length >= 5 && content.trim().length >= 10 && selectedCategory !== '';
            this.setData({
                canSubmit
            });
        },

        // 提交问题
        onSubmit() {
            if (!this.canSubmit || this.isSubmitting) {
                return;
            }
            const { title, content, selectedCategory, images, isAnonymous } = this;

            // 表单验证
            if (title.trim().length < 5) {
                uni.showToast({
                    title: '标题至少需要5个字',
                    icon: 'none'
                });
                return;
            }
            if (content.trim().length < 10) {
                uni.showToast({
                    title: '描述至少需要10个字',
                    icon: 'none'
                });
                return;
            }
            if (!selectedCategory) {
                uni.showToast({
                    title: '请选择问题分类',
                    icon: 'none'
                });
                return;
            }
            this.setData({
                isSubmitting: true
            });

            // 如果有图片，先上传图片
            if (images.length > 0) {
                this.uploadImages(images)
                    .then((imageUrls) => {
                        this.createQuestion(title, content, selectedCategory, imageUrls, isAnonymous);
                    })
                    .catch((err) => {
                        console.error('图片上传失败:', err);
                        uni.showToast({
                            title: '图片上传失败',
                            icon: 'none'
                        });
                        this.setData({
                            isSubmitting: false
                        });
                    });
            } else {
                this.createQuestion(title, content, selectedCategory, [], isAnonymous);
            }
        },

        // 上传图片
        uploadImages(images) {
            return new Promise((resolve, reject) => {
                const uploadPromises = images.map((imagePath) => {
                    return new Promise((resolveUpload, rejectUpload) => {
                        uni.uploadFile({
                            url: 'https://your-api-endpoint.com/upload',
                            filePath: imagePath,
                            name: 'file',
                            success: (res) => {
                                const data = JSON.parse(res.data);
                                resolveUpload(data.url);
                            },
                            fail: rejectUpload
                        });
                    });
                });
                Promise.all(uploadPromises)
                    .then((urls) => resolve(urls))
                    .catch((err) => reject(err));
            });
        },

        // 创建问题
        createQuestion(title, content, category, images, isAnonymous) {
            // 模拟提交
            setTimeout(() => {
                const categoryNames = {
                    training: '训练',
                    health: '健康',
                    diet: '饮食',
                    behavior: '行为'
                };
                const newQuestion = {
                    id: `qa_${Date.now()}`,
                    title: title,
                    content: content,
                    category: category,
                    categoryName: categoryNames[category],
                    images: images,
                    author: {
                        id: isAnonymous ? 'anonymous' : 'user_current',
                        nickname: isAnonymous ? '匿名用户' : '当前用户',
                        avatar: isAnonymous
                            ? 'https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?w=100&h=100&fit=crop'
                            : 'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=100&h=100&fit=crop'
                    },
                    stats: {
                        views: 0,
                        answerCount: 0,
                        follows: 0
                    },
                    isAnonymous: isAnonymous,
                    isFollowing: false,
                    createdAt: '刚刚'
                };

                // 实际开发时调用API
                // wx.request({
                //   url: `${app.globalData.baseUrl}/questions`,
                //   method: 'POST',
                //   data: {
                //     title,
                //     content,
                //     category,
                //     images,
                //     isAnonymous
                //   },
                //   success: (res) => {
                //     wx.showToast({ title: '发布成功', icon: 'success' });
                //     setTimeout(() => {
                //       wx.navigateBack();
                //     }, 1500);
                //   },
                //   fail: () => {
                //     wx.showToast({ title: '发布失败', icon: 'none' });
                //   },
                //   complete: () => {
                //     this.setData({ isSubmitting: false });
                //   }
                // });

                uni.showToast({
                    title: '发布成功',
                    icon: 'success'
                });
                setTimeout(() => {
                    // 返回并刷新列表
                    const pages = getCurrentPages();
                    const discoverPage = pages.find((p) => p.route === 'pages/discover/discover');
                    if (discoverPage) {
                        discoverPage.setData({
                            questions: [newQuestion, ...discoverPage.data.questions]
                        });
                    }
                    uni.navigateBack();
                }, 1500);
                this.setData({
                    isSubmitting: false
                });
            }, 1500);
        },

        // 返回上一页
        onCancel() {
            if (this.title || this.content || this.images.length > 0) {
                uni.showModal({
                    title: '确认退出',
                    content: '退出后已填写的内容将不会保存',
                    confirmText: '退出',
                    confirmColor: '#ff4757',
                    cancelText: '继续编辑',
                    success: (res) => {
                        if (res.confirm) {
                            uni.navigateBack();
                        }
                    }
                });
            } else {
                uni.navigateBack();
            }
        }
    }
};
</script>
<style>
@import './question-create.css';
</style>
