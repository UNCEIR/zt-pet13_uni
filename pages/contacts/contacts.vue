<template>
    <view>
        <!-- 通讯录页面 -->
        <view class="page">
            <!-- 搜索框 -->
            <view class="search-card">
                <view class="search-box">
                    <view class="search-icon-svg">
                        <view class="svg-search"></view>
                    </view>
                    <input class="search-input" type="text" placeholder="搜索联系人" :value="searchKeyword" @input="onSearchInput" confirm-type="search" />
                    <view class="clear-btn" v-if="searchKeyword" @tap="onSearchClear">
                        <view class="clear-icon">×</view>
                    </view>
                </view>
            </view>

            <!-- 新的成员入口 -->
            <view class="menu-card" @tap="onNewMemberTap">
                <view class="menu-item">
                    <view class="new-member-icon">
                        <text class="new-member-icon-text">+</text>
                    </view>
                    <text class="menu-text">新的成员</text>
                    <view class="arrow"></view>
                </view>
            </view>

            <!-- 成员列表 -->
            <view class="contacts-list">
                <block v-if="!isEmpty">
                    <view class="letter-group" :id="'group-' + item.letter" v-for="(item, index) in groupedMembers" :key="index">
                        <!-- 分组标题 -->

                        <view class="group-header">{{ item.letter }}</view>

                        <!-- 成员项 -->

                        <view class="group-content">
                            <view
                                class="member-item"
                                :data-id="member.id"
                                :data-member="member"
                                @tap="onMemberTap"
                                @longpress="onMemberLongPress"
                                v-for="(member, index1) in item.list"
                                :key="index1"
                            >
                                <!-- 头像 -->

                                <view class="member-avatar">
                                    <image v-if="member.avatar" :src="member.avatar" mode="aspectFill" />
                                    <text v-else class="avatar-text">{{ member.name[0] }}</text>
                                </view>

                                <!-- 信息 -->

                                <view class="member-info">
                                    <view class="member-name-row">
                                        <text class="member-name">{{ member.name }}</text>
                                        <text :class="'role-tag ' + member.roleType">{{ member.role }}</text>
                                    </view>
                                    <view class="member-phone">{{ member.phone }}</view>
                                </view>

                                <!-- 箭头 -->

                                <view class="arrow"></view>
                            </view>
                        </view>
                    </view>
                </block>

                <!-- 空状态 -->
                <view class="empty-state" v-if="isEmpty">
                    <view class="empty-icon-svg">
                        <view class="svg-contacts"></view>
                    </view>
                    <text class="empty-text">暂无联系人</text>
                    <text class="empty-tip">点击右上角添加新成员</text>
                </view>
            </view>

            <!-- 右侧字母导航 -->
            <view class="letter-sidebar" v-if="!isEmpty && !searchKeyword">
                <view
                    :class="'letter-item ' + (currentLetter === item ? 'active' : '')"
                    :data-letter="item"
                    @tap="onLetterTap"
                    @touchstart="onLetterTouchStart"
                    v-for="(item, index) in letters"
                    :key="index"
                >
                    {{ item }}
                </view>
            </view>

            <!-- 字母提示浮层 -->
            <view class="letter-toast" v-if="showLetterToast">
                <text class="letter-toast-text">{{ currentLetter }}</text>
            </view>
        </view>

        <!-- 添加按钮（悬浮） -->
        <view class="add-btn-float" @tap="onAddTap" v-if="false">
            <text class="add-icon">+</text>
        </view>
    </view>
</template>

<script>
// 通讯录页面逻辑
export default {
    data() {
        return {
            // 搜索关键词
            searchKeyword: '',

            // 是否显示字母提示
            showLetterToast: false,

            // 当前高亮字母
            currentLetter: '',

            // 字母表
            letters: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '#'],

            // 原始成员数据
            members: [],

            // 分组后的成员数据
            groupedMembers: [],

            // 是否显示空状态
            isEmpty: false,

            member: {
                id: '',
                avatar: '',
                name: '',
                roleType: '',
                role: '',
                phone: ''
            }
        };
    },
    onLoad() {
        this.loadMockData();
    },
    onShow() {
        // 刷新数据
        this.loadMockData();
    },
    // 下拉刷新
    onPullDownRefresh() {
        this.loadMockData();
        uni.stopPullDownRefresh();
    },
    methods: {
        // 加载模拟数据
        loadMockData() {
            const mockMembers = [
                {
                    id: '1',
                    name: '阿伟',
                    avatar: '',
                    phone: '139****5678',
                    role: '主人',
                    roleType: 'owner',
                    pinyin: 'awei',
                    firstLetter: 'A',
                    petName: '旺财'
                },
                {
                    id: '2',
                    name: '阿姨',
                    avatar: '',
                    phone: '138****1234',
                    role: '主人',
                    roleType: 'owner',
                    pinyin: 'ayi',
                    firstLetter: 'A',
                    petName: '旺财'
                },
                {
                    id: '3',
                    name: '表弟',
                    avatar: '',
                    phone: '136****9876',
                    role: '家庭成员',
                    roleType: 'family',
                    pinyin: 'biaodi',
                    firstLetter: 'B',
                    petName: '旺财'
                },
                {
                    id: '4',
                    name: '陈小姐',
                    avatar: '',
                    phone: '135****2468',
                    role: '临时看护',
                    roleType: 'temp',
                    pinyin: 'chenxiaojie',
                    firstLetter: 'C',
                    petName: '来福'
                },
                {
                    id: '5',
                    name: '大壮',
                    avatar: '',
                    phone: '137****1357',
                    role: '家庭成员',
                    roleType: 'family',
                    pinyin: 'dazhuang',
                    firstLetter: 'D',
                    petName: '来福'
                },
                {
                    id: '6',
                    name: '芳芳',
                    avatar: '',
                    phone: '150****8642',
                    role: '临时看护',
                    roleType: 'temp',
                    pinyin: 'fangfang',
                    firstLetter: 'F',
                    petName: '咪咪'
                },
                {
                    id: '7',
                    name: '隔壁老王',
                    avatar: '',
                    phone: '151****9753',
                    role: '临时看护',
                    roleType: 'temp',
                    pinyin: 'gebilaowang',
                    firstLetter: 'G',
                    petName: '旺财'
                },
                {
                    id: '8',
                    name: '表姐',
                    avatar: '',
                    phone: '152****3579',
                    role: '家庭成员',
                    roleType: 'family',
                    pinyin: 'biaojie',
                    firstLetter: 'B',
                    petName: '咪咪'
                },
                {
                    id: '9',
                    name: '小李',
                    avatar: '',
                    phone: '153****7410',
                    role: '临时看护',
                    roleType: 'temp',
                    pinyin: 'xiaoli',
                    firstLetter: 'X',
                    petName: '来福'
                },
                {
                    id: '10',
                    name: '张医生',
                    avatar: '',
                    phone: '155****2580',
                    role: '临时看护',
                    roleType: 'temp',
                    pinyin: 'zhangyisheng',
                    firstLetter: 'Z',
                    petName: '所有宠物'
                }
            ];
            this.setData({
                members: mockMembers
            });
            this.groupMembers(mockMembers);
        },

        // 按首字母分组
        groupMembers(members) {
            const groups = {};
            const letters = [];

            // 按首字母分组
            members.forEach((member) => {
                const letter = member.firstLetter || '#';
                if (!groups[letter]) {
                    groups[letter] = [];
                    letters.push(letter);
                }
                groups[letter].push(member);
            });

            // 字母排序
            letters.sort();

            // 构建分组数据
            const groupedMembers = letters.map((letter) => ({
                letter,
                list: groups[letter]
            }));
            this.setData({
                groupedMembers,
                isEmpty: groupedMembers.length === 0
            });
        },

        // 搜索输入
        onSearchInput(e) {
            const keyword = e.detail.value.toLowerCase();
            this.setData({
                searchKeyword: keyword
            });
            this.filterMembers(keyword);
        },

        // 搜索清除
        onSearchClear() {
            this.setData({
                searchKeyword: ''
            });
            this.filterMembers('');
        },

        // 过滤成员
        filterMembers(keyword) {
            if (!keyword) {
                this.groupMembers(this.members);
                return;
            }
            const filtered = this.members.filter((member) => {
                return member.name.toLowerCase().includes(keyword) || member.pinyin.toLowerCase().includes(keyword) || member.phone.includes(keyword);
            });
            this.groupMembers(filtered);
        },

        // 点击字母导航
        onLetterTap(e) {
            const letter = e.currentTarget.dataset.letter;
            this.setData({
                showLetterToast: true,
                currentLetter: letter
            });

            // 滚动到对应分组
            uni.createSelectorQuery()
                .in(uni)
                .select(`#group-${letter}`)
                .boundingClientRect((rect) => {
                    if (rect) {
                        uni.pageScrollTo({
                            selector: `#group-${letter}`,
                            duration: 300
                        });
                    }
                })
                .exec();

            // 1秒后隐藏字母提示
            setTimeout(() => {
                this.setData({
                    showLetterToast: false
                });
            }, 1000);
        },

        // 触摸字母导航（滑动选择）
        onLetterTouchStart(e) {
            const letter = e.currentTarget.dataset.letter;
            this.onLetterTap({
                currentTarget: {
                    dataset: {
                        letter
                    }
                }
            });
        },

        // 点击成员项
        onMemberTap(e) {
            const memberId = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: `/pages/member-detail/member-detail?id=${memberId}`
            });
        },

        // 长按成员项（删除）
        onMemberLongPress(e) {
            const member = e.currentTarget.dataset.member;
            uni.showModal({
                title: '删除成员',
                content: `确定删除 ${member.name} 吗？`,
                confirmColor: '#ff4444',
                success: (res) => {
                    if (res.confirm) {
                        this.deleteMember(member.id);
                    }
                }
            });
        },

        // 删除成员
        deleteMember(memberId) {
            const members = this.members.filter((m) => m.id !== memberId);
            this.setData({
                members
            });
            this.groupMembers(members);
            uni.showToast({
                title: '删除成功',
                icon: 'success'
            });
        },

        // 点击添加按钮
        onAddTap() {
            uni.navigateTo({
                url: '/pages/add-member/add-member'
            });
        },

        // 点击新的成员
        onNewMemberTap() {
            uni.navigateTo({
                url: '/pages/add-member/add-member'
            });
        }
    }
};
</script>
<style>
@import './contacts.css';
</style>
