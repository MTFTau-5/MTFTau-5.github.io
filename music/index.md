---
title: 音乐
layout: page
---

<div id="music-player"></div>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.js"></script>

<script>
  // 强化路径配置（解决404问题）
// 强制修正路径（解决拼写错误）
  const repoName = 'mtftau-5.github.io'; // 严格匹配仓库名
  const isOnline = window.location.host.includes('github.io');
  const basePath = isOnline ? `/${repoName.toLowerCase()}` : '';

  // 添加容错加载逻辑
  function loadMusic() {
    const fallbackUrls = [
      `${basePath}/music.json`,
      'https://raw.githubusercontent.com/MTFTau-5/mtftau-5.github.io/main/source/music.json'
    ];

    for (const url of fallbackUrls) {
      try {
        const res = await fetch(url);
        if (res.ok) return res.json();
      } catch (e) {
        console.warn(`尝试 ${url} 失败:`, e);
      }
    }
    throw new Error('所有备用加载方式均失败');
  }
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