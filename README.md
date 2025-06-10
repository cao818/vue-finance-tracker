# Vue3 记账 / 收支管理系统

这是一个基于 **Vue3 + Vite + Element Plus** 开发的简单实用的记账/收支管理系统，支持收入支出记录管理、分类汇总、图表分析、数据导入导出等功能，适合用于学习 Vue3 项目开发与练习。

## 🌟 项目特点

- 📌 支持收入/支出记录增删改查
- 📊 图表展示当前收支情况（使用 ECharts）
- 📂 分类汇总统计表格
- 🔍 筛选功能：按类型、分类关键词、日期范围筛选记录
- 💾 本地存储（LocalStorage）保存数据
- 📥 数据导入 / 📤 导出 JSON 文件
- 🌐 支持部署到 GitHub Pages

## 📦 技术栈

- [Vue 3](https://vuejs.org/)
- [Vite](https://vitejs.dev/)
- [Element Plus](https://element-plus.org/)
- [ECharts](https://echarts.apache.org/)
- [GitHub Pages](https://pages.github.com/)

## 📸 项目预览

> 👉 演示地址：[https://yourname.github.io/vue-finance-tracker/](https://cao818.github.io/vue-finance-tracker/)

![demo](./public/demo-screenshot.png)

## 项目结构
```bash
.
├── public/ # 静态资源
├── src/
│ ├── components/ # 组件
│ ├── App.vue # 根组件
│ └── main.js # 入口文件
├── dist/ # 构建输出目录
├── vite.config.js # Vite 配置文件
├── package.json # 项目配置及依赖
└── README.md # 项目说明
```
## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/cao818/vue-finance-tracker.git
cd vue-finance-tracker
```
### 2. 安装依赖
```bash
npm install
```
### 3.本地运行
```bash
npm run dev
```

## 构建与部署
### 1.配置 vite.config.js
```bash
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  base: '/vue-finance-tracker/', // 替换成你的仓库名
  plugins: [vue()]
})
```
### 2. 构建项目
```bash
npm run build
```
### 3. 安装 gh-pages（用于发布）
```bash
npm install -D gh-pages
```
### 4. 在 package.json 添加部署脚本
```bash
"scripts": {
  "deploy": "gh-pages -d dist"
}
```
### 5. 发布到 GitHub Pages
```bash
npm run deploy
```
### 6. 在 GitHub 仓库设置中启用 GitHub Pages
- 选择 gh-pages 分支，根目录 /

- 保存并等待几分钟即可访问：
https://yourname.github.io/vue-finance-tracker/

## 数据导入导出说明
- 导入和导出格式为 JSON 文件。

- 单条记录结构示例：
```json
{
  "id": "1654854654876",
  "type": "income",      // 或 "expense"
  "category": "工资",
  "amount": 5000,
  "date": "2025-06-10"
}
```








