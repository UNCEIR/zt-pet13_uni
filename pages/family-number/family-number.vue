<template>
    <view class="page">
        <!-- 顶部说明区 -->
        <view class="fn-header">
            <view class="fn-header__icon">
                <view class="fn-header__phone-ring">
                    <view class="ring ring--outer"></view>
                    <view class="ring ring--inner"></view>
                    <view class="phone-body"></view>
                </view>
            </view>
            <view class="fn-header__content">
                <text class="fn-header__title">亲情号管理</text>
                <text class="fn-header__desc">设置设备端快速拨号号码，仅亲情号可呼入设备</text>
            </view>

            <view class="fn-header__right">
                <view class="fn-header__count">
                    <text class="fn-header__count-num">{{ filledCount }}</text>
                    <text class="fn-header__count-sep">/</text>
                    <text class="fn-header__count-total">5</text>
                </view>
                <view :class="'fn-header__sync fn-header__sync--' + syncStatus">
                    <view class="sync-dot"></view>
                    <text class="sync-text" v-if="syncStatus === 'synced'">已同步</text>
                    <text class="sync-text" v-else-if="syncStatus === 'syncing'">同步中...</text>
                    <text class="sync-text" v-else>同步失败</text>
                </view>
            </view>
        </view>

        <!-- 槽位列表 -->
        <view class="fn-slots">
            <block v-for="(item, index) in slots" :key="index">
                <!-- 已填充槽位 -->

                <view class="fn-slot fn-slot--filled" v-if="item" :data-index="index" @tap="onSlotTap" @longpress="onSlotLongPress">
                    <view class="fn-slot__badge">
                        <text class="fn-slot__badge-num">{{ index + 1 }}</text>
                    </view>
                    <view class="fn-slot__info">
                        <text class="fn-slot__name">{{ item.name }}</text>
                        <text class="fn-slot__phone">{{ item.phone }}</text>
                    </view>
                    <view class="fn-slot__controls">
                        <view class="fn-slot__perms">
                            <view :class="'perm-tag perm-tag--' + (item.callIn ? 'on' : 'off')">
                                <text class="perm-tag__text">呼入</text>
                            </view>
                            <view :class="'perm-tag perm-tag--' + (item.callOut ? 'on' : 'off')">
                                <text class="perm-tag__text">呼出</text>
                            </view>
                        </view>
                        <view :class="'fn-slot__sync-icon fn-slot__sync-icon--' + (item.synced ? 'ok' : 'pending')">
                            <view class="sync-check" v-if="item.synced"></view>
                            <view class="sync-spinner" v-else></view>
                        </view>
                    </view>
                    <view class="fn-slot__arrow"></view>
                </view>

                <!-- 空槽位 -->

                <view class="fn-slot fn-slot--empty" v-else :data-index="index" @tap="onSlotTap">
                    <view class="fn-slot__badge fn-slot__badge--empty">
                        <text class="fn-slot__badge-num">{{ index + 1 }}</text>
                    </view>
                    <view class="fn-slot__add">
                        <view class="fn-slot__add-icon">+</view>
                        <text class="fn-slot__add-text">添加亲情号</text>
                    </view>
                </view>
            </block>
        </view>

        <!-- 底部提示 -->
        <view class="fn-footer">
            <text class="fn-footer__tip">长按已有号码可编辑或删除，号码修改后将自动同步至设备</text>
        </view>

        <!-- 添加/编辑弹窗 -->
        <view class="fn-modal" v-if="showModal" @tap="onModalMaskTap">
            <view class="fn-modal__card" catch:tap>
                <view class="fn-modal__handle"></view>
                <view class="fn-modal__header">
                    <text class="fn-modal__title">{{ editingSlot >= 0 && slots[editingSlot] ? '编辑' : '添加' }}亲情号 #{{ editingSlot + 1 }}</text>
                </view>

                <!-- 快捷标签 -->
                <view class="fn-modal__labels">
                    <view
                        :class="'label-chip ' + (selectedLabel === item ? 'label-chip--active' : '')"
                        :data-label="item"
                        @tap="onLabelTap"
                        v-for="(item, index) in quickLabels"
                        :key="index"
                    >
                        <text class="label-chip__text">{{ item }}</text>
                    </view>
                </view>

                <!-- 备注名输入 -->
                <view class="fn-modal__field">
                    <text class="fn-modal__field-label">备注名</text>
                    <input class="fn-modal__input" :value="modalData.name" placeholder="输入备注名称" maxlength="10" @input="onNameInput" />
                </view>

                <!-- 手机号输入 -->
                <view class="fn-modal__field">
                    <text class="fn-modal__field-label">手机号</text>
                    <input class="fn-modal__input" type="number" :value="modalData.phone" placeholder="输入11位手机号码" maxlength="11" @input="onPhoneInput" />
                </view>

                <!-- 权限开关 -->
                <view class="fn-modal__switches">
                    <view class="fn-modal__switch-row">
                        <view class="fn-modal__switch-info">
                            <text class="fn-modal__switch-label">允许呼入</text>
                            <text class="fn-modal__switch-desc">该号码可拨打设备</text>
                        </view>
                        <switch :checked="modalData.callIn" color="#89E04E" @change="onModalCallInChange" />
                    </view>
                    <view class="fn-modal__switch-row">
                        <view class="fn-modal__switch-info">
                            <text class="fn-modal__switch-label">允许呼出</text>
                            <text class="fn-modal__switch-desc">设备可拨打该号码</text>
                        </view>
                        <switch :checked="modalData.callOut" color="#89E04E" @change="onModalCallOutChange" />
                    </view>
                </view>

                <!-- 操作按钮 -->
                <view class="fn-modal__actions">
                    <view class="fn-modal__btn fn-modal__btn--cancel" @tap="onModalCancel">
                        <text>取消</text>
                    </view>
                    <view class="fn-modal__btn fn-modal__btn--confirm" @tap="onModalConfirm">
                        <text>确认保存</text>
                    </view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            slots: [null, null, null, null, null],
            showModal: false,
            editingSlot: -1,
            modalData: {
                name: '',
                phone: '',
                callIn: true,
                callOut: true
            },
            syncStatus: 'synced',
            filledCount: 0,
            quickLabels: ['爸爸', '妈妈', '家人', '朋友', '其他'],
            selectedLabel: ''
        };
    },
    onLoad() {
        this.loadSlots();
        this.updateSlotCount();
    },
    methods: {
        loadSlots() {
            const device = app.globalData._selectedDevice;
            const imei = device ? device.imei : app.globalData.defaultDeviceImei || '';
            const key = `family_numbers_${imei}`;
            const saved = uni.getStorageSync(key);
            if (saved && saved.length) {
                const slots = [null, null, null, null, null];
                saved.forEach((item) => {
                    if (item && item.slot >= 1 && item.slot <= 5) {
                        slots[item.slot - 1] = item;
                    }
                });
                this.setData({
                    slots
                });
            } else {
                this.setData({
                    slots: [
                        {
                            slot: 1,
                            name: '爸爸',
                            phone: '13812345678',
                            callIn: true,
                            callOut: true,
                            synced: true
                        },
                        {
                            slot: 2,
                            name: '妈妈',
                            phone: '13987654321',
                            callIn: true,
                            callOut: false,
                            synced: true
                        },
                        null,
                        null,
                        null
                    ]
                });
            }
        },

        saveSlots() {
            const device = app.globalData._selectedDevice;
            const imei = device ? device.imei : app.globalData.defaultDeviceImei || '';
            const key = `family_numbers_${imei}`;
            const toSave = this.slots.filter((s) => s !== null);
            uni.setStorageSync(key, toSave);
        },

        maskPhone(phone) {
            if (!phone || phone.length < 7) {
                return phone;
            }
            return phone.substring(0, 3) + '****' + phone.substring(7);
        },

        onSlotTap(e) {
            const idx = e.currentTarget.dataset.index;
            const slot = this.slots[idx];
            if (slot) {
                this.setData({
                    showModal: true,
                    editingSlot: idx,
                    selectedLabel: slot.name,
                    modalData: {
                        name: slot.name,
                        phone: slot.phone,
                        callIn: slot.callIn,
                        callOut: slot.callOut
                    }
                });
            } else {
                this.setData({
                    showModal: true,
                    editingSlot: idx,
                    selectedLabel: '',
                    modalData: {
                        name: '',
                        phone: '',
                        callIn: true,
                        callOut: true
                    }
                });
            }
        },

        onSlotLongPress(e) {
            const idx = e.currentTarget.dataset.index;
            const slot = this.slots[idx];
            if (!slot) {
                return;
            }
            uni.showActionSheet({
                itemList: ['编辑', '删除'],
                success: (res) => {
                    if (res.tapIndex === 0) {
                        this.onSlotTap(e);
                    } else if (res.tapIndex === 1) {
                        this.deleteSlot(idx);
                    }
                }
            });
        },

        deleteSlot(idx) {
            uni.showModal({
                title: '确认删除',
                content: `确定删除亲情号 ${this.slots[idx].name}？`,
                confirmColor: '#2E7D32',
                success: (res) => {
                    if (res.confirm) {
                        this.setData({
                            [`slots[${idx}]`]: null
                        });
                        this.saveSlots();
                        this.updateSlotCount();
                        this.simulateSync();
                        uni.showToast({
                            title: '已删除',
                            icon: 'success'
                        });
                    }
                }
            });
        },

        onCallInChange(e) {
            const idx = e.currentTarget.dataset.index;
            this.setData({
                [`slots[${idx}].callIn`]: e.detail.value
            });
            this.saveSlots();
            this.simulateSync();
        },

        onCallOutChange(e) {
            const idx = e.currentTarget.dataset.index;
            this.setData({
                [`slots[${idx}].callOut`]: e.detail.value
            });
            this.saveSlots();
            this.simulateSync();
        },

        onLabelTap(e) {
            const label = e.currentTarget.dataset.label;
            this.setData({
                selectedLabel: label,
                'modalData.name': label
            });
        },

        onNameInput(e) {
            this.setData({
                'modalData.name': e.detail.value,
                selectedLabel: ''
            });
        },

        onPhoneInput(e) {
            this.setData({
                'modalData.phone': e.detail.value
            });
        },

        onModalCallInChange(e) {
            this.setData({
                'modalData.callIn': e.detail.value
            });
        },

        onModalCallOutChange(e) {
            this.setData({
                'modalData.callOut': e.detail.value
            });
        },

        onModalConfirm() {
            const { name, phone, callIn, callOut } = this.modalData;
            const idx = this.editingSlot;
            if (!name.trim()) {
                uni.showToast({
                    title: '请输入备注名',
                    icon: 'none'
                });
                return;
            }
            if (!/^1[3-9]\d{9}$/.test(phone)) {
                uni.showToast({
                    title: '请输入正确的手机号',
                    icon: 'none'
                });
                return;
            }
            const slotData = {
                slot: idx + 1,
                name: name.trim(),
                phone,
                callIn,
                callOut,
                synced: false
            };
            this.setData({
                [`slots[${idx}]`]: slotData,
                showModal: false,
                editingSlot: -1
            });
            this.saveSlots();
            this.updateSlotCount();
            this.simulateSync();
        },

        onModalCancel() {
            this.setData({
                showModal: false,
                editingSlot: -1,
                selectedLabel: ''
            });
        },

        onModalMaskTap() {
            this.onModalCancel();
        },

        updateSlotCount() {
            const count = this.slots.filter((s) => s !== null).length;
            this.setData({
                filledCount: count
            });
        },

        simulateSync() {
            this.setData({
                syncStatus: 'syncing'
            });
            const filledIndices = [];
            this.slots.forEach((s, i) => {
                if (s !== null) {
                    filledIndices.push(i);
                    this.setData({
                        [`slots[${i}].synced`]: false
                    });
                }
            });
            setTimeout(() => {
                filledIndices.forEach((i) => {
                    if (this.slots[i]) {
                        this.setData({
                            [`slots[${i}].synced`]: true
                        });
                    }
                });
                this.setData({
                    syncStatus: 'synced'
                });
                this.saveSlots();
                this.updateSlotCount();
            }, 1500);
        }
    }
};
</script>
<style>
@import './family-number.css';
</style>
