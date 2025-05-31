---
title: 音乐
layout: page
---

<div id="music-player"></div>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.js"></script>

<script>
  // 强化路径配置（解决404问题）
  const repoName = 'MTFTau-5.github.io'; // 必须小写
  const isGitHubPages = window.location.host.includes('github.io');
  const basePath = isGitHubPages ? `/${repoName.toLowerCase()}` : ''; // GitHub强制小写路径

  // 调试信息
  console.log('当前路径基准:', basePath);
  
  fetch(`${basePath}/music.json?t=${Date.now()}`) // 添加时间戳避免缓存
    .then(async response => {
      console.log('HTTP状态码:', response.status);
      
      // 严格检查响应类型
      if (response.status === 404) {
        throw new Error(`music.json 文件不存在于 ${basePath}/music.json`);
      }
      
      const data = await response.text();
      try {
        return JSON.parse(data); // 防止非JSON内容
      } catch (e) {
        console.error('实际返回内容:', data.substring(0, 100));
        throw new Error('服务器返回了非JSON数据（可能是404页面）');
      }
    })
    .then(musicList => {
      if (!Array.isArray(musicList)) {
        throw new Error('music.json 内容必须是数组');
      }

      // 初始化播放器（带路径验证）
      new APlayer({
        container: document.getElementById('music-player'),
        theme: '#F57474',
        audio: musicList.map(file => {
          if (!file.url) throw new Error('缺少必填字段: url');
          
          // 自动修复路径格式
          const cleanUrl = file.url.startsWith('/') ? file.url : `/${file.url}`;
          return {
            name: file.name || '未命名',
            artist: file.artist || '未知艺术家',
            url: `${basePath}${cleanUrl}`.replace(/([^:]\/)\/+/g, '$1'), // 去重斜杠
            cover: file.cover ? `${basePath}/${file.cover}`.replace('//', '/') : ''
          };
        })
      });
    })
    .catch(error => {
      console.error('完整错误:', error);
      document.getElementById('music-player').innerHTML = `
        <div style="color:red; padding:1em; background:#ffeeee;">
          <h4>⚠️ 音乐加载失败</h4>
          <p><strong>具体错误:</strong> ${error.message}</p>
          <hr>
          <p><strong>请按以下步骤排查:</strong></p>
          <ol>
            <li>访问 <a href="${basePath}/music.json" target="_blank">${basePath}/music.json</a> 确认文件存在</li>
            <li>检查仓库中 <code>music.json</code> 是否在根目录</li>
            <li>确保 Hexo 配置中已添加: <code>skip_render: ['music.json']</code></li>
          </ol>
          <p>按 F12 打开控制台查看网络请求详情</p>
        </div>
      `;
    });
</script>