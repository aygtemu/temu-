# temuSan插件联网更新功能

本文件夹包含temuSan插件的联网更新功能实现。通过GitHub Pages免费托管，可以实现插件的版本检查、词库更新等功能。

## 文件结构

- `version_update.js`: 版本检查和更新模块
- `word_update.js`: 词库更新模块
- `update.html`: 更新界面
- `update.js`: 更新界面交互逻辑
- `github_pages_templates.zip`: GitHub Pages服务器文件模板（虚拟ZIP）

## 使用方法

### 1. 配置GitHub Pages服务器

1. 创建一个GitHub仓库（可以是公开的）
2. 在仓库根目录创建以下文件：
   - `version.json`: 版本信息文件
   - `health.json`: 服务器健康状态文件
   - `words.json`: 词库文件
   - `changelog.json`: 更新日志文件
   - `changelog.html`: 更新日志网页
3. 启用GitHub Pages（Settings → Pages → 选择main分支）

### 2. 修改插件配置

1. 在`update.js`中修改`UPDATE_SERVER_URL`为你的GitHub Pages地址：
   ```javascript
   const UPDATE_SERVER_URL = 'https://您的用户名.github.io/仓库名';
   ```

2. 在`version_update.js`中修改默认的更新服务器URL：
   ```javascript
   updateServerUrl: 'https://您的用户名.github.io/仓库名'
   ```

3. 在`word_update.js`中修改默认的更新服务器URL：
   ```javascript
   updateServerUrl: 'https://您的用户名.github.io/仓库名'
   ```

### 3. 集成到插件中

1. 修改`manifest.json`，添加主机权限：
   ```json
   "host_permissions": [
     "https://您的用户名.github.io/仓库名/*"
   ]
   ```

2. 在background.js中添加定期检查更新的代码：
   ```javascript
   // 导入更新模块
   import { initUpdateChecker } from './network_update/version_update.js';
   
   // 初始化更新检查器
   initUpdateChecker({
     updateServerUrl: 'https://您的用户名.github.io/仓库名',
     checkInterval: 24 * 60 * 60 * 1000, // 24小时
     notifyOnUpdate: true
   });
   ```

3. 在gallery.js中添加联网检测按钮的完整实现：
   ```javascript
   // 导入词库更新模块
   import { performNetworkCheckAndUpdate } from './network_update/word_update.js';
   
   // 修改setupWordActions函数中的联网检测按钮处理
   function setupWordActions() {
     // ... 现有代码 ...
     
     // 联网检测按钮
     document.getElementById('check-network-btn').addEventListener('click', async () => {
       const button = this;
       const originalText = button.textContent;
       button.disabled = true;
       button.textContent = '检测中...';
       
       try {
         // 获取当前词库
         const { prohibitedWords, filterWords } = await getLocalWordSets();
         
         // 执行检查和更新
         const result = await performNetworkCheckAndUpdate({
           updateServerUrl: 'https://您的用户名.github.io/仓库名',
           currentProhibitedWords: prohibitedWords,
           currentFilterWords: filterWords,
           onUpdateProgress: (progress) => {
             console.log('更新进度:', progress);
           }
         });
         
         if (result.success) {
           let message = '联网检测成功！';
           
           if (result.hasNewVersion) {
             message += ` 发现新版本 ${result.versionUpdateInfo.version}！`;
           }
           
           if (result.wordUpdateResult) {
             const wordResult = result.wordUpdateResult;
             if (wordResult.oldProhibitedCount !== wordResult.newProhibitedCount) {
               message += `\n违禁词库更新: ${wordResult.oldProhibitedCount} → ${wordResult.newProhibitedCount}`;
             }
             if (wordResult.oldFilterCount !== wordResult.newFilterCount) {
               message += `\n过滤词库更新: ${wordResult.oldFilterCount} → ${wordResult.newFilterCount}`;
             }
           }
           
           showNotification('成功', message);
         } else {
           showNotification('错误', result.error || '联网检测失败');
         }
       } catch (error) {
         showNotification('错误', '操作失败: ' + error.message);
       } finally {
         button.disabled = false;
         button.textContent = originalText;
       }
     });
     
     // ... 现有代码 ...
   }
   ```

### 4. 添加更新页面到插件

在插件的popup页面或设置页面中添加更新中心的链接：
```html
<a href="network_update/update.html" target="_blank">更新中心</a>
```

## 功能特点

1. **自动版本检查**：定期检查插件是否有新版本
2. **词库在线更新**：支持违禁词和过滤词的在线更新
3. **服务器健康检测**：确保更新服务器可用
4. **友好的更新界面**：显示版本信息、词库数量和更新日志
5. **更新通知**：发现新版本时显示浏览器通知

## 注意事项

1. 确保GitHub Pages正确配置，并且所有JSON文件格式正确
2. 定期更新服务器上的词库和版本信息
3. 注意CORS问题，GitHub Pages默认支持跨域请求
4. 考虑为敏感词库设置访问限制（如果需要）

## 免费替代方案

如果GitHub Pages不适合，可以考虑以下免费替代方案：

1. **Firebase Hosting**：Google提供的免费静态托管服务
2. **Netlify**：提供免费的静态网站托管
3. **Vercel**：专注于前端应用的免费托管平台

这些服务都提供免费的静态网站托管，可以用来部署更新服务器。
