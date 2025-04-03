# Vue3 照片墙项目

这是一个使用Vue3开发的3D照片墙项目，可以展示和浏览照片集。项目已配置为可以自动部署到GitHub Pages。

## 项目特点

- 3D旋转照片墙效果
- 自动播放和手动控制功能
- 响应式设计，适配移动设备
- 支持键盘控制（左右箭头切换，空格暂停/播放）

## 开发环境设置

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview
```

## GitHub Pages 部署

本项目已配置为使用GitHub Actions自动部署到GitHub Pages。每当推送到main分支时，GitHub Actions会自动构建项目并部署到GitHub Pages。

### GitHub Pages 配置步骤

1. 进入GitHub仓库设置页面：`https://github.com/用户名/仓库名/settings/pages`
2. 在「Build and deployment」部分：
   - Source: 选择「GitHub Actions」
   - 确保已启用GitHub Pages功能

### 自动部署配置

- 项目的`vite.config.js`中已设置`base`为`/`（已更新）
- `.github/workflows/deploy.yml`文件配置了自动构建和部署流程

### 常见部署问题解决

如果遇到部署错误（如404错误），请检查：

1. 确认GitHub Pages已在仓库设置中正确启用
2. 确认仓库名称格式是否为`用户名.github.io`
   - 如果是，`vite.config.js`中的`base`应设为`/`
   - 如果不是，`vite.config.js`中的`base`应设为`/仓库名/`

### 手动部署

如果需要手动部署，可以执行以下步骤：

```bash
# 构建项目
npm run build

# 将dist目录推送到GitHub Pages分支
```

## 项目结构

- `src/components/PhotoWall.vue` - 3D照片墙组件
- `src/img/` - 存放照片的目录
- `src/main.js` - 应用入口文件

## 自定义

要添加或更新照片，只需将新照片放入`src/img/`目录，项目会自动导入并显示。