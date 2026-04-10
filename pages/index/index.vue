<template>
    <view>
        <!-- 宠物首页 - 名宠 Tab -->
        <view class="page">
            <!-- 沉浸式主视觉区：仅背景+头部，无任何功能区 -->
            <view class="hero-section">
                <image class="hero-bg" src="/static/images/hero-bg-pure.jpg" mode="aspectFill" />
                <view class="hero-header" :style="'padding-top: ' + headerTop + 'px;'">
                    <view class="header">
                        <view class="header-left">
                            <view class="avatar-wrap">
                                <image class="avatar" src="/static/images/pet-avatar.png" mode="aspectFill" />
                                <view class="status-dot online"></view>
                            </view>
                            <view class="pet-info">
                                <text class="pet-name">名宠</text>
                                <text class="pet-status">在线</text>
                            </view>
                        </view>
                        <view class="header-right">
                            <image class="icon-status" :src="'/static/images/icons/' + (bluetoothConnected ? '蓝牙连接成功' : '蓝牙未连接') + '.png'" mode="aspectFit" />
                            <image class="icon-status" src="/static/images/icons/设备在线.png" mode="aspectFit" />
                            <image class="icon-status battery" src="/static/images/icons/电量.png" mode="aspectFit" />
                            <view class="msg-wrap" @tap="goMessage">
                                <image class="icon-status" src="/static/images/icons/未读消息.png" mode="aspectFit" />
                                <view class="msg-badge"></view>
                            </view>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 按钮详情 -->
            <view class="quick-actions">
                <view class="quick-btn" @tap="onRing">
                    <image class="quick-icon" src="/static/images/icons/响铃.png" mode="aspectFit" />
                    <text class="quick-text">响铃</text>
                </view>
                <view class="quick-btn disabled">
                    <image class="quick-icon" src="/static/images/icons/寻宠.png" mode="aspectFit" />
                    <text class="quick-text">寻宠</text>
                </view>
                <view class="quick-btn disabled">
                    <image class="quick-icon" src="/static/images/icons/语音互动.svg" mode="aspectFit" />
                    <text class="quick-text">语音互动</text>
                </view>
            </view>

            <!-- 四大核心功能 -->
            <view class="card-section">
                <!-- 第一行：左 电子围栏 | 右 轨迹回放 -->
                <view class="core-row">
                    <view class="core-card">
                        <view class="core-header" @tap="goFence">
                            <image class="core-icon" src="/static/images/icons/热点围栏.png" mode="aspectFit" />
                            <text class="core-title">电子围栏</text>
                            <view class="arrow"></view>
                        </view>
                        <view class="core-subs">
                            <view class="sub-btn" @tap="goFenceInOut">
                                <image class="sub-icon" src="/static/images/icons/出入围栏.png" mode="aspectFit" />
                                <text>出入围栏</text>
                            </view>
                            <view class="sub-btn" @tap="goFenceHotspot">
                                <image class="sub-icon" :src="'/static/images/icons/热点围栏.png'" mode="aspectFit" />
                                <text>热点围栏</text>
                            </view>
                        </view>
                    </view>
                    <view class="core-card">
                        <view class="core-header" @tap="goTrack">
                            <image class="core-icon" src="/static/images/icons/轨迹回放.png" mode="aspectFit" />
                            <text class="core-title">轨迹回放</text>
                            <view class="arrow"></view>
                        </view>
                        <view class="core-subs">
                            <view class="sub-btn" @tap="goTrackToday">
                                <image class="sub-icon" src="/static/images/icons/历史轨迹.png" mode="aspectFit" />
                                <text>今日轨迹</text>
                            </view>
                            <view class="sub-btn" @tap="goTrackHistory">
                                <image class="sub-icon" src="/static/images/icons/历史轨迹.png" mode="aspectFit" />
                                <text>历史轨迹</text>
                            </view>
                        </view>
                    </view>
                </view>
                <!-- 第二行：左 健康数据 | 右 多宠管理 -->
                <view class="core-row">
                    <view class="core-card">
                        <view class="core-header" @tap="goHealth">
                            <image class="core-icon" src="/static/images/icons/活动.png" mode="aspectFit" />
                            <text class="core-title">健康数据</text>
                            <view class="arrow"></view>
                        </view>
                        <view class="core-data">
                            <view class="data-card">
                                <image class="data-icon" src="/static/images/icons/活动.png" mode="aspectFit" />
                                <text class="data-value">1h 30min</text>
                                <text class="data-label">活动</text>
                            </view>
                            <view class="data-card">
                                <image class="data-icon" src="/static/images/icons/睡眠.png" mode="aspectFit" />
                                <text class="data-value">3h 20min</text>
                                <text class="data-label">睡眠</text>
                            </view>
                        </view>
                    </view>
                    <view class="core-card">
                        <view class="core-header" @tap="goPetList">
                            <image class="core-icon" src="/static/images/icons/添加萌宠.png" mode="aspectFit" />
                            <text class="core-title">多宠管理</text>
                            <view class="arrow"></view>
                        </view>
                        <view class="core-subs">
                            <view class="sub-btn" @tap="goAddOwnerOrMember">
                                <image class="sub-icon" src="/static/images/icons/添加主人.png" mode="aspectFit" />
                                <text>{{ hasOwner ? '添加成员' : '添加主人' }}</text>
                            </view>
                            <view class="sub-btn" @tap="goAddPet">
                                <image class="sub-icon" src="/static/images/icons/添加萌宠.png" mode="aspectFit" />
                                <text>添加萌宠</text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>
        </view>

        <!-- 响铃执行中弹框 -->
        <view class="modal-overlay" v-if="showRingModal" @touchmove.stop.prevent="preventMove">
            <view class="modal-content modal-dark">
                <view class="modal-loading"></view>
                <text class="modal-text">正在执行指令中.....</text>
            </view>
        </view>

        <!-- 响铃成功弹框 -->
        <view class="modal-overlay" v-if="showSuccessModal" @tap="closeSuccessModal" @touchmove.stop.prevent="preventMove">
            <view class="modal-content modal-dark success" @tap.stop.prevent="stopPropagation">
                <view class="modal-icon-success">
                    <view class="modal-check"></view>
                </view>
                <text class="modal-text">设备已响铃</text>
                <view class="modal-btn" @tap="closeSuccessModal">确认</view>
            </view>
        </view>

        <!-- 蓝牙未开启底部弹窗 -->
        <view class="bs-overlay" v-if="showBtOffPopup" @tap="dismissBtOffPopup" @touchmove.stop.prevent="preventMove">
            <view class="bs-panel" @tap.stop.prevent="stopPropagation">
                <view class="bs-head">
                    <image class="bs-head-icon" src="/static/images/icons/蓝牙未连接.png" mode="aspectFit" />
                    <text class="bs-head-title">蓝牙未开启</text>
                    <view class="bs-head-close" @tap="dismissBtOffPopup">×</view>
                </view>
                <view class="bs-body-card">
                    <text class="bs-desc">蓝牙未开启，无法通过蓝牙使用感应开锁等用车功能</text>
                    <text class="bs-hint">进入控制中心，点击蓝牙图标</text>
                    <view class="cc-mock">
                        <view class="cc-grid">
                            <!-- 飞行模式 -->
                            <view class="cc-dot cc-off">
                                <view class="cc-airplane">
                                    <view class="ap-body"></view>
                                    <view class="ap-wing"></view>
                                    <view class="ap-tail"></view>
                                </view>
                            </view>
                            <!-- 蜂窝数据 -->
                            <view class="cc-dot cc-off">
                                <view class="cc-cellular">
                                    <view class="cell-bar" style="height: 8rpx"></view>
                                    <view class="cell-bar" style="height: 14rpx"></view>
                                    <view class="cell-bar" style="height: 20rpx"></view>
                                    <view class="cell-bar" style="height: 26rpx"></view>
                                </view>
                            </view>
                            <!-- WiFi -->
                            <view class="cc-dot cc-on-green">
                                <view class="cc-wifi">
                                    <view class="wifi-arc wifi-lg"></view>
                                    <view class="wifi-arc wifi-md"></view>
                                    <view class="wifi-arc wifi-sm"></view>
                                    <view class="wifi-center"></view>
                                </view>
                            </view>
                            <!-- 蓝牙 -->
                            <view class="cc-dot cc-on-blue">
                                <view class="cc-bt">
                                    <view class="bt-line"></view>
                                    <view class="bt-arrow-t"></view>
                                    <view class="bt-arrow-b"></view>
                                </view>
                            </view>
                        </view>
                    </view>
                </view>
                <view class="bs-action" @tap="closeBtOffPopup">我知道了</view>
            </view>
        </view>

        <!-- 添加附近蓝牙底部弹窗 -->
        <view class="bs-overlay" v-if="showBtListPopup" @tap="closeBtListPopup" @touchmove.stop.prevent="preventMove">
            <view class="bs-panel bs-panel-list" @tap.stop.prevent="stopPropagation">
                <view class="bs-head">
                    <view style="width: 44rpx"></view>
                    <text class="bs-head-title">添加附近蓝牙</text>
                    <view class="bs-head-close" @tap="closeBtListPopup">×</view>
                </view>
                <view class="bs-body">
                    <text class="bs-label">已添加蓝牙</text>
                    <view class="dev-card">
                        <view class="dev-item" v-for="(item, index) in addedDevices" :key="index">
                            <view class="dev-check">✓</view>

                            <text class="dev-name">{{ item.name }}</text>

                            <view class="dev-delete" @tap="removeDevice" :data-index="index">
                                <view class="trash-body"></view>
                            </view>
                        </view>
                        <view class="dev-empty" v-if="addedDevices.length === 0">暂无已添加设备</view>
                    </view>
                    <text class="bs-label">附近寻呼机</text>
                    <view class="dev-card dev-card-gray">
                        <view
                            :class="'dev-item nearby ' + (item.selected ? 'selected' : '')"
                            @tap="selectDevice"
                            :data-index="index"
                            v-for="(item, index) in nearbyDevices"
                            :key="index"
                        >
                            <text class="dev-name">{{ item.name }}</text>

                            <view class="dev-bt-sym">
                                <view class="bts-line"></view>
                                <view class="bts-top"></view>
                                <view class="bts-bot"></view>
                            </view>
                        </view>
                    </view>
                </view>
                <view :class="'bs-action ' + (selectedDeviceIndex < 0 ? 'bs-action-disabled' : '')" @tap="confirmBtConnect">确定</view>
            </view>
        </view>

        <!-- 热点围栏成功/取消弹窗 -->
        <view class="modal-overlay" v-if="showHotspotSuccessModal" @tap="closeHotspotModal" @touchmove.stop.prevent="preventMove">
            <view class="modal-content modal-dark success" @tap.stop.prevent="stopPropagation">
                <view class="modal-icon-success">
                    <view class="modal-check"></view>
                </view>
                <text class="modal-text">{{ hotspotModalText }}</text>
                <view class="modal-btn" @tap="closeHotspotModal">确认</view>
            </view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            hasOwner: false,
            showRingModal: false,
            showSuccessModal: false,
            headerTop: 54,
            bluetoothConnected: false,
            hotspotEnabled: false,
            showBtOffPopup: false,
            showBtListPopup: false,
            showHotspotSuccessModal: false,
            hotspotModalText: '',
            addedDevices: [
                {
                    name: '小米音响蓝牙'
                }
            ],
            nearbyDevices: [
                {
                    name: 'Baseus Encok W04',
                    selected: false
                },
                {
                    name: 'EDIFIER LolliPods R',
                    selected: false
                },
                {
                    name: 'GP01-099355',
                    selected: false
                }
            ],
            selectedDeviceIndex: -1
        };
    },
    onLoad() {
        const sys = uni.getSystemInfoSync();
        const statusBarHeight = sys.statusBarHeight || 44;
        const hasOwner = app.globalData.getHasOwner();
        this.setData({
            hasOwner,
            headerTop: statusBarHeight + 10,
            bluetoothConnected: false,
            hotspotEnabled: false
        });
    },
    onShow() {
        const hasOwner = app.globalData.getHasOwner();
        this.setData({
            hasOwner
        });
    },
    methods: {
        // 响铃交互
        onRing() {
            this.setData({
                showRingModal: true
            });
            setTimeout(() => {
                this.setData({
                    showRingModal: false,
                    showSuccessModal: true
                });
            }, 1500);
        },

        closeSuccessModal() {
            this.setData({
                showSuccessModal: false
            });
        },

        preventMove() {},
        stopPropagation() {},

        // 蓝牙弹窗交互
        dismissBtOffPopup() {
            this.setData({
                showBtOffPopup: false
            });
        },

        closeBtOffPopup() {
            this.setData({
                showBtOffPopup: false,
                showBtListPopup: true
            });
        },

        closeBtListPopup() {
            this.setData({
                showBtListPopup: false
            });
        },

        selectDevice(e) {
            const idx = e.currentTarget.dataset.index;
            const nearbyDevices = this.nearbyDevices;
            nearbyDevices.forEach((d, i) => {
                d.selected = i === idx;
            });
            this.setData({
                nearbyDevices,
                selectedDeviceIndex: idx
            });
        },

        removeDevice(e) {
            const idx = e.currentTarget.dataset.index;
            const addedDevices = this.addedDevices;
            addedDevices.splice(idx, 1);
            this.setData({
                addedDevices
            });
        },

        confirmBtConnect() {
            if (this.selectedDeviceIndex < 0) {
                return;
            }
            const idx = this.selectedDeviceIndex;
            const device = this.nearbyDevices[idx];
            const addedDevices = this.addedDevices.concat([
                {
                    name: device.name
                }
            ]);
            const nearbyDevices = this.nearbyDevices.filter((_, i) => i !== idx);
            this.setData({
                bluetoothConnected: true,
                showBtListPopup: false,
                addedDevices,
                nearbyDevices,
                selectedDeviceIndex: -1
            });
        },

        closeHotspotModal() {
            this.setData({
                showHotspotSuccessModal: false
            });
        },

        // 跳转
        goMessage() {
            uni.navigateTo({
                url: '/pages/message/message'
            });
        },

        goFence() {
            uni.navigateTo({
                url: '/pages/fence/fence'
            });
        },

        goTrack() {
            uni.navigateTo({
                url: '/pages/track/track'
            });
        },

        goPetList() {
            uni.navigateTo({
                url: '/pages/pet-list/pet-list'
            });
        },

        goHealth() {
            uni.navigateTo({
                url: '/pages/health/health'
            });
        },

        goFenceInOut() {
            uni.navigateTo({
                url: '/pages/fence-in-out/fence-in-out'
            });
        },

        goFenceHotspot() {
            if (!this.bluetoothConnected) {
                this.setData({
                    showBtOffPopup: true
                });
                return;
            }
            const enabling = !this.hotspotEnabled;
            this.setData({
                hotspotEnabled: enabling,
                showHotspotSuccessModal: true,
                hotspotModalText: enabling ? '已启用热点围栏' : '已取消热点围栏'
            });
        },

        goTrackToday() {
            uni.navigateTo({
                url: '/pages/track-today/track-today'
            });
        },

        goTrackHistory() {
            uni.navigateTo({
                url: '/pages/track-history/track-history'
            });
        },

        goAddOwner() {
            uni.navigateTo({
                url: '/pages/add-owner/add-owner'
            });
        },

        goAddOwnerOrMember() {
            if (this.hasOwner) {
                uni.navigateTo({
                    url: '/pages/add-member/add-member'
                });
                return;
            }
            uni.navigateTo({
                url: '/pages/add-owner/add-owner'
            });
        },

        goAddMember() {
            uni.navigateTo({
                url: '/pages/add-member/add-member'
            });
        },

        goAddPet() {
            uni.navigateTo({
                url: '/pages/add-pet/add-pet'
            });
        }
    }
};
</script>
<style>
@import './index.css';
</style>
