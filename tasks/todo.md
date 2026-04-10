# 亲情号功能页设计与实现

## 完成清单

- [x] 创建 `pages/family-number/` 页面四件套（wxml/wxss/js/json）
- [x] 在 `app.json` 注册页面路由
- [x] 修改 `pages/mine/mine.js` 的 `onFamilyNumber()` 跳转逻辑
- [x] 精调视觉细节：渐变、动画、同步状态指示器、拖拽指示条
- [x] 检查 lint 错误，确认所有文件无报错

## 变更摘要

### 新建文件
| 文件 | 说明 |
|------|------|
| `pages/family-number/family-number.wxml` | 页面模板：顶部说明卡、5 槽位列表、底部提示、添加/编辑弹窗 |
| `pages/family-number/family-number.wxss` | 页面样式：绿色渐变系统、卡片/徽章/弹窗/动画 |
| `pages/family-number/family-number.js` | 页面逻辑：槽位 CRUD、本地存储、模拟同步、校验 |
| `pages/family-number/family-number.json` | 页面配置 |

### 修改文件
| 文件 | 变更 |
|------|------|
| `app.json` | 添加 `pages/family-number/family-number` 路由 |
| `pages/mine/mine.js` | `onFamilyNumber()` 改为 `wx.navigateTo` 跳转 |

## Review

- 所有触控区域 >= 88rpx (44px)，满足 touch-target-size 规范
- BEM 命名规范（fn-header__icon, fn-slot--filled 等）
- 使用 data path 局部更新（`slots[0].callIn`）提升 setData 性能
- 动画时长 200-300ms 范围内，符合微交互最佳实践
- 无 lint 错误
