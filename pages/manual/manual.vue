<template>
    <view class="page">
        <!-- 顶部说明区 -->
        <view class="manual-header">
            <view class="manual-header__icon">
                <view class="manual-header__book">
                    <view class="book-spine"></view>
                    <view class="book-pages"></view>
                </view>
            </view>
            <view class="manual-header__content">
                <text class="manual-header__title">电子说明书</text>
                <text class="manual-header__desc">快速了解设备功能与使用方法</text>
            </view>
            <view class="manual-header__device">
                <text class="manual-header__model">ZT-PET13</text>
                <text class="manual-header__version">智能定位器</text>
            </view>
        </view>

        <!-- 功能导航区 -->
        <view class="manual-nav">
            <view class="manual-nav__item" :data-id="item.id" @tap="onNavTap" v-for="(item, index) in navItems" :key="index">
                <view :class="'manual-nav__icon manual-nav__icon--' + item.icon">
                    <view class="icon-inner"></view>
                </view>

                <text class="manual-nav__text">{{ item.title }}</text>
            </view>
        </view>

        <!-- 内容展示区 - 折叠面板 -->
        <view class="manual-content">
            <!-- 产品功能介绍 -->
            <view class="manual-section">
                <view class="manual-section__header" data-section="features" @tap="toggleSection">
                    <view class="manual-section__title-wrapper">
                        <view class="manual-section__icon manual-section__icon--star"></view>
                        <text class="manual-section__title">产品功能介绍</text>
                    </view>
                    <view :class="'manual-section__arrow ' + (expandedSections.features ? 'manual-section__arrow--up' : '')"></view>
                </view>
                <view class="manual-section__body" v-if="expandedSections.features">
                    <view class="feature-list">
                        <view class="feature-item" v-for="(item, index) in features" :key="index">
                            <view :class="'feature-item__icon feature-item__icon--' + item.icon"></view>

                            <view class="feature-item__content">
                                <text class="feature-item__title">{{ item.title }}</text>
                                <text class="feature-item__desc">{{ item.desc }}</text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 快速开始指南 -->
            <view class="manual-section">
                <view class="manual-section__header" data-section="quickstart" @tap="toggleSection">
                    <view class="manual-section__title-wrapper">
                        <view class="manual-section__icon manual-section__icon--rocket"></view>
                        <text class="manual-section__title">快速开始指南</text>
                    </view>
                    <view :class="'manual-section__arrow ' + (expandedSections.quickstart ? 'manual-section__arrow--up' : '')"></view>
                </view>
                <view class="manual-section__body" v-if="expandedSections.quickstart">
                    <view class="step-list">
                        <view class="step-item" v-for="(item, index) in quickStartSteps" :key="index">
                            <view class="step-item__num">{{ item.num }}</view>

                            <view class="step-item__content">
                                <text class="step-item__title">{{ item.title }}</text>
                                <text class="step-item__desc">{{ item.desc }}</text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 常见问题FAQ -->
            <view class="manual-section">
                <view class="manual-section__header" data-section="faq" @tap="toggleSection">
                    <view class="manual-section__title-wrapper">
                        <view class="manual-section__icon manual-section__icon--faq"></view>
                        <text class="manual-section__title">常见问题FAQ</text>
                    </view>
                    <view :class="'manual-section__arrow ' + (expandedSections.faq ? 'manual-section__arrow--up' : '')"></view>
                </view>
                <view class="manual-section__body" v-if="expandedSections.faq">
                    <view class="faq-list">
                        <view class="faq-item" :data-index="index" @tap="toggleFaq" v-for="(item, index) in faqList" :key="index">
                            <view class="faq-item__question">
                                <text class="faq-item__q">Q：</text>
                                <text class="faq-item__text">{{ item.question }}</text>
                                <view :class="'faq-item__arrow ' + (item.expanded ? 'faq-item__arrow--up' : '')"></view>
                            </view>

                            <view class="faq-item__answer" v-if="item.expanded">
                                <text class="faq-item__a">A：</text>
                                <text class="faq-item__answer-text">{{ item.answer }}</text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 设备操作指南 -->
            <view class="manual-section">
                <view class="manual-section__header" data-section="device" @tap="toggleSection">
                    <view class="manual-section__title-wrapper">
                        <view class="manual-section__icon manual-section__icon--device"></view>
                        <text class="manual-section__title">设备操作指南</text>
                    </view>
                    <view :class="'manual-section__arrow ' + (expandedSections.device ? 'manual-section__arrow--up' : '')"></view>
                </view>
                <view class="manual-section__body" v-if="expandedSections.device">
                    <view class="guide-list">
                        <view class="guide-item" :data-index="index" @tap="toggleDeviceGuide" v-for="(item, index) in deviceGuides" :key="index">
                            <view class="guide-item__header">
                                <text class="guide-item__title">{{ item.title }}</text>
                                <view :class="'guide-item__arrow ' + (item.expanded ? 'guide-item__arrow--up' : '')"></view>
                            </view>

                            <view class="guide-item__content" v-if="item.expanded">
                                <text class="guide-item__text">{{ item.content }}</text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>

            <!-- 安全注意事项 -->
            <view class="manual-section">
                <view class="manual-section__header" data-section="safety" @tap="toggleSection">
                    <view class="manual-section__title-wrapper">
                        <view class="manual-section__icon manual-section__icon--safety"></view>
                        <text class="manual-section__title">安全注意事项</text>
                    </view>
                    <view :class="'manual-section__arrow ' + (expandedSections.safety ? 'manual-section__arrow--up' : '')"></view>
                </view>
                <view class="manual-section__body" v-if="expandedSections.safety">
                    <view class="safety-list">
                        <view class="safety-item" :data-index="index" @tap="toggleSafety" v-for="(item, index) in safetyItems" :key="index">
                            <view class="safety-item__header">
                                <view class="safety-item__dot"></view>
                                <text class="safety-item__title">{{ item.title }}</text>
                                <view :class="'safety-item__arrow ' + (item.expanded ? 'safety-item__arrow--up' : '')"></view>
                            </view>

                            <view class="safety-item__content" v-if="item.expanded">
                                <text class="safety-item__text">{{ item.content }}</text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>
        </view>

        <!-- 底部区域 -->
        <view class="manual-footer">
            <view class="manual-footer__support">
                <view class="manual-footer__item" @tap="onCallSupport">
                    <view class="manual-footer__icon manual-footer__icon--phone"></view>
                    <text class="manual-footer__label">客服电话</text>
                    <text class="manual-footer__value">{{ supportPhone }}</text>
                </view>
                <view class="manual-footer__divider"></view>
                <view class="manual-footer__item" @tap="onOnlineSupport">
                    <view class="manual-footer__icon manual-footer__icon--chat"></view>
                    <text class="manual-footer__label">在线客服</text>
                    <text class="manual-footer__value">点击咨询</text>
                </view>
            </view>
            <view class="manual-footer__version">
                <text>说明书版本 {{ version }}</text>
            </view>
        </view>
    </view>
</template>

<script>
const app = getApp();
export default {
    data() {
        return {
            // 折叠面板展开状态
            expandedSections: {
                features: true,
                // 产品功能默认展开
                quickstart: false,
                faq: false,
                device: false,
                safety: false
            },
            // 功能导航列表
            navItems: [
                {
                    id: 'quickstart',
                    icon: 'book',
                    title: '快速入门'
                },
                {
                    id: 'features',
                    icon: 'location',
                    title: '定位功能'
                },
                {
                    id: 'fence',
                    icon: 'fence',
                    title: '围栏设置'
                },
                {
                    id: 'track',
                    icon: 'track',
                    title: '轨迹查询'
                },
                {
                    id: 'health',
                    icon: 'health',
                    title: '健康监测'
                },
                {
                    id: 'sos',
                    icon: 'sos',
                    title: 'SOS功能'
                }
            ],
            // 产品功能列表
            features: [
                {
                    icon: 'location',
                    title: '实时定位',
                    desc: 'GPS+北斗+基站三重定位'
                },
                {
                    icon: 'track',
                    title: '历史轨迹',
                    desc: '30天轨迹回放'
                },
                {
                    icon: 'fence',
                    title: '电子围栏',
                    desc: '安全区域设置与提醒'
                },
                {
                    icon: 'health',
                    title: '健康监测',
                    desc: '运动步数、睡眠分析'
                },
                {
                    icon: 'sos',
                    title: 'SOS求助',
                    desc: '一键紧急呼叫'
                },
                {
                    icon: 'call',
                    title: '双向通话',
                    desc: '设备与APP语音通话'
                }
            ],
            // 快速开始步骤
            quickStartSteps: [
                {
                    num: 1,
                    title: '设备开机与充电',
                    desc: '长按电源键3秒开机，使用原装充电器充电'
                },
                {
                    num: 2,
                    title: 'APP绑定设备',
                    desc: '扫描设备背面二维码或手动输入IMEI号'
                },
                {
                    num: 3,
                    title: '插入SIM卡',
                    desc: '使用标准Nano SIM卡，确保开通流量和语音'
                },
                {
                    num: 4,
                    title: '设置亲情号',
                    desc: '在"我的-亲情号"中添加可通话号码'
                },
                {
                    num: 5,
                    title: '激活电子围栏',
                    desc: '设置安全区域，超出范围自动报警'
                }
            ],
            // FAQ列表
            faqList: [
                {
                    question: '设备无法定位怎么办？',
                    answer: '请检查：1. 设备已开机且有电量；2. SIM卡已开通流量；3. 设备在室外或信号良好区域；4. APP已绑定正确设备。如仍无法定位，请重启设备或联系客服。'
                },
                {
                    question: '如何查看历史轨迹？',
                    answer: '在首页点击"历史轨迹"，选择要查看的日期，即可查看当天的活动轨迹。支持查看最近30天的轨迹记录。'
                },
                {
                    question: '围栏报警不推送？',
                    answer: '请检查：1. 手机通知权限已开启；2. APP在后台运行；3. 围栏功能已开启且设置正确；4. 设备电量充足。可在"设置-消息通知"中检查通知权限。'
                },
                {
                    question: '设备续航时间多久？',
                    answer: '正常使用续航约3-5天，具体取决于定位频率和使用场景。开启省电模式可延长至7天。建议每天充电或按APP低电量提醒及时充电。'
                },
                {
                    question: '如何恢复出厂设置？',
                    answer: '在设备关机状态下，同时长按电源键和SOS键10秒，听到提示音后松开，设备将恢复出厂设置。注意：恢复出厂设置会清除所有数据，请谨慎操作。'
                }
            ],
            // 设备操作指南
            deviceGuides: [
                {
                    title: '按键功能说明',
                    content: '电源键：长按3秒开机/关机，短按查看电量\nSOS键：长按3秒触发紧急呼叫\n音量键：调节通话音量'
                },
                {
                    title: '指示灯状态',
                    content: '绿灯常亮：正常工作\n绿灯闪烁：定位中\n红灯闪烁：电量低\n蓝灯闪烁：充电中\n黄灯闪烁：无信号/SIM卡异常'
                },
                {
                    title: '充电注意事项',
                    content: '1. 使用原装充电器\n2. 充电时请保持干燥\n3. 充满电约2小时\n4. 避免过度充电'
                },
                {
                    title: 'SOS呼叫流程',
                    content: '1. 长按SOS键3秒\n2. 设备依次拨打亲情号\n3. APP同步收到推送通知\n4. 可远程监听设备周围声音'
                }
            ],
            // 安全注意事项
            safetyItems: [
                {
                    title: '防水等级',
                    content: '设备支持IP67级防水，可在雨天使用，但请勿长时间浸泡或游泳时佩戴。'
                },
                {
                    title: '适用温度',
                    content: '工作温度：-10°C 至 50°C\n充电温度：0°C 至 45°C\n极端温度可能影响电池性能。'
                },
                {
                    title: 'SIM卡要求',
                    content: '请使用中国移动/联通/电信Nano SIM卡，确保开通2G/4G流量和语音通话功能。'
                },
                {
                    title: '设备保养',
                    content: '1. 定期清洁设备表面\n2. 避免强烈撞击\n3. 远离强磁场\n4. 长期不用时保持50%电量存放'
                }
            ],
            // 底部信息
            supportPhone: '400-888-8888',
            version: 'V1.2.0'
        };
    },
    onLoad() {
        // 页面加载时的初始化
    },
    methods: {
        // 切换折叠面板
        toggleSection(e) {
            const section = e.currentTarget.dataset.section;
            const expandedSections = {
                ...this.expandedSections
            };
            expandedSections[section] = !expandedSections[section];
            this.setData({
                expandedSections
            });
        },

        // 功能导航点击
        onNavTap(e) {
            const id = e.currentTarget.dataset.id;
            const expandedSections = {
                ...this.expandedSections
            };

            // 关闭所有面板
            Object.keys(expandedSections).forEach((key) => {
                expandedSections[key] = false;
            });

            // 展开对应面板
            expandedSections[id] = true;
            this.setData({
                expandedSections
            });

            // 滚动到对应区域
            this.scrollToSection(id);
        },

        // 滚动到指定区域
        scrollToSection(id) {
            // 微信小程序中可以通过 selectComponent 或 scroll-view 实现滚动
            // 这里使用简单的展开逻辑，实际滚动可由用户手动完成
        },

        // FAQ展开/收起
        toggleFaq(e) {
            const index = e.currentTarget.dataset.index;
            const faqList = [...this.faqList];
            faqList[index].expanded = !faqList[index].expanded;
            this.setData({
                faqList
            });
        },

        // 设备指南展开/收起
        toggleDeviceGuide(e) {
            const index = e.currentTarget.dataset.index;
            const deviceGuides = [...this.deviceGuides];
            deviceGuides[index].expanded = !deviceGuides[index].expanded;
            this.setData({
                deviceGuides
            });
        },

        // 安全事项展开/收起
        toggleSafety(e) {
            const index = e.currentTarget.dataset.index;
            const safetyItems = [...this.safetyItems];
            safetyItems[index].expanded = !safetyItems[index].expanded;
            this.setData({
                safetyItems
            });
        },

        // 拨打客服电话
        onCallSupport() {
            uni.makePhoneCall({
                phoneNumber: this.supportPhone
            });
        },

        // 在线客服
        onOnlineSupport() {
            uni.showToast({
                title: '客服功能即将上线',
                icon: 'none'
            });
        }
    }
};
</script>
<style>
@import './manual.css';
</style>
