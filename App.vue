<script>
// zt-pet13 宠物定位微信小程序
export default {
    data() {
        return {};
    },
    globalData: {
        // 默认设备参数
        defaultDeviceImei: '862506074976309',

        defaultDeviceModel: 'PT13',

        // 当前宠物示例信息
        defaultPet: {
            name: '喵喵',
            gender: '公',
            birthYear: 2025,
            birthMonth: 6,
            breed: '美短猫'
        },

        // 是否已绑定创建者（主人）
        hasOwner: false,

        // 当前主人名称
        ownerName: '',

        // 宠物列表
        pets: [],

        // 设备列表（可绑定）
        devices: [
            {
                imei: '862506074976309',
                model: 'PT13'
            },
            {
                imei: '862506074976310',
                model: 'PT13'
            },
            {
                imei: '862506074976311',
                model: 'PT13'
            },
            {
                imei: '862506074976312',
                model: 'PT14'
            },
            {
                imei: '862506074976313',
                model: 'PT13'
            },
            {
                imei: '862506074976314',
                model: 'PT14'
            }
        ],

        getHasOwner() {
            return this.hasOwner;
        }
    },
    onLaunch() {
        //模拟器重新加载后清空登录缓存，再次显示「添加主人」
        this.globalData.hasOwner = false;
        this.globalData.ownerName = '';
        uni.removeStorageSync('hasOwner');
        uni.removeStorageSync('hasOwnerExpireAt');
        uni.removeStorageSync('ownerName');

        // 从本地存储恢复其余状态（不恢复 hasOwner/ownerName）
        const pets = uni.getStorageSync('pets');
        if (pets && Array.isArray(pets)) {
            this.globalData.pets = pets;
        }
    }
};
</script>
<style>
@import './app.css';
</style>
