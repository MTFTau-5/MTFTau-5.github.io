---
title: 音乐
layout: page
---

<div id="music-player"></div>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.js"></script>

<script>
  // 异步加载音乐数据
  (async function initPlayer() {
    try {
      // 动态路径配置（修复GitHub Pages路径问题）
      const repoName = 'MTFTau-5.github.io';
      const isGitHubPages = window.location.host.includes('github.io');
      const basePath = isGitHubPages ? `/${repoName}` : '';

      // 加载音乐列表
      const response = await fetch(`${basePath}/music.json`);
      if (!response.ok) throw new Error(`HTTP错误: ${response.status}`);
      
      const musicList = await response.json();
      if (!Array.isArray(musicList)) throw new Error('music.json格式应为数组');

      // 初始化播放器
      new APlayer({
        container: document.getElementById('music-player'),
        theme: '#F57474',
        audio: musicList.map(file => ({
          name: file.name?.replace(/\.[^/.]+$/, '') || '未命名',
          artist: file.artist || '未知艺术家',
          url: `${basePath}${file.url.startsWith('/') ? '' : '/'}${file.url}`,
          cover: file.cover ? `${basePath}/${file.cover}`.replace('//', '/') : ''
        }))
      });

    } catch (error) {
      console.error('播放器初始化失败:', error);
      document.getElementById('music-player').innerHTML = `
        <div style="color: red; padding: 1em; border: 1px dashed red;">
          <p>播放器加载失败</p>
          <p>原因: ${error.message}</p>
          <p>请检查: </p>
          <ol>
            <li>music.json文件是否存在？</li>
            <li>浏览器控制台是否有其他报错？</li>
          </ol>
        </div>
      `;
    }
  })();
</script>