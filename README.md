# ✈️ Travel Planner - 旅行规划工具集

智能旅行规划功能模块集合，使用 TypeScript 构建。

## 📦 模块列表

| 模块 | 路径 | 说明 |
|------|------|------|
| 🧘 冥想放松引导 | `src/meditation/` | 旅行场景冥想播放器 + 音频缓存 |
| 🎵 当地音乐发现 | `src/music/` | 目的地音乐风格与艺术家推荐 |
| 🏃 跑步路线推荐 | `src/running/` | 全球城市经典跑步路线数据 |
| 📚 读书清单生成 | `src/books/` | 目的地相关书籍推荐 |
| ☕ 咖啡店探索 | `src/coffee/` | 全球特色咖啡店指南 |
| 🧳 智能打包清单 | `src/packing/` | 基于气候×活动的行李清单生成 |
| 💰 预算追踪器 | `src/budget/` | 多币种支出记录 + 汇率转换 |
| 🆘 紧急联系卡片 | `src/emergency/` | 5国紧急电话/大使馆/求救短语 |
| 🌅 日出日落追踪 | `src/sunrise/` | NOAA算法计算日出日落时间 |
| 🎉 节日日历 | `src/festival/` | 全球传统节日数据 |
| 👕 天气穿搭建议 | `src/outfit/` | 温度/湿度/风力穿搭推荐引擎 |
| 🥗 素食餐厅指南 | `src/vegan/` | 素食/纯素餐厅推荐 |
| 🗣️ 语言速查手册 | `src/phrasebook/` | 常用旅行短语（含发音） |
| 📶 WiFi/SIM卡指南 | `src/connectivity/` | 各目的地网络方案对比 |
| 📸 摄影地点推荐 | `src/photography/` | 经典拍摄机位与时段 |

## 🚀 使用

```typescript
// 生成打包清单
import { generateChecklist } from './src/packing/engine/generator';
const list = generateChecklist('东京', 5, 'temperate', ['cultural']);

// 计算日出日落
import { calculateSunTimes } from './src/sunrise/engine/calculator';
const times = calculateSunTimes(35.6762, 139.6503, new Date());

// 穿搭建议
import { recommendOutfit } from './src/outfit/engine/recommender';
const outfit = recommendOutfit({ tempC: 15, humidity: 60, windKmh: 10, rain: false, uv: 3 });
```

## 📄 License

MIT
