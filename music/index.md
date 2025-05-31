---
title: 音乐
layout: page
---

<div id="music-player"></div>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.js"></script>

<script>
  // 强化版音乐播放器初始化
  (async function() {
    // 调试标记
    console.log('[Debug] 开始初始化播放器');
    
    try {
      // 动态路径配置（兼容所有环境）
      const repoName = 'mtftau-5.github.io'.toLowerCase(); // 强制小写
      const isOnline = /github\.io$/i.test(window.location.host);
      const basePath = isOnline ? `/${repoName}` : '';
      
      // 三重保险加载方式
      const musicData = await loadMusicWithFallback(basePath);
      
      // 初始化播放器（带路径消毒）
      initAPlayer(musicData, basePath);

    } catch (error) {
      showError(error);
    }

    // 核心函数定义
    async function loadMusicWithFallback(basePath) {
      const fallbackUrls = [
        `${basePath}/music.json`,
        `${basePath}/music.json?t=${Date.now()}`,
        'https://raw.githubusercontent.com/MTFTau-5/mtftau-5.github.io/main/source/music.json'
      ];

      for (const url of fallbackUrls) {
        try {
          const res = await fetch(url);
          if (!res.ok) continue;
          
          const data = await res.json();
          if (Array.isArray(data)) {
            console.log('[Success] 音乐数据加载自:', url);
            return data;
          }
        } catch (e) {
          console.warn(`[Fallback] 尝试 ${url} 失败:`, e);
        }
      }
      throw new Error('所有备用加载方式均失败');
    }

    function initAPlayer(musicList, basePath) {
      // 路径消毒函数
      const sanitizePath = (path) => {
        return `${basePath}/${path}`
          .replace(/([^:]\/)\/+/g, '$1') // 去重斜杠
          .replace(/[^\w./-]/g, '_');    // 替换特殊字符
      };

      new APlayer({
        container: document.getElementById('music-player'),
        theme: '#F57474',
        audio: musicList.map(item => ({
          name: item.name || '未命名曲目',
          artist: item.artist || '未知艺术家',
          url: sanitizePath(item.url),
          cover: item.cover ? sanitizePath(item.cover) : ''
        }))
      });
      console.log('[Success] APlayer初始化完成');
    }

    function showError(error) {
      console.error('[Error] 播放器初始化失败:', error);
      const debugInfo = {
        timestamp: new Date().toISOString(),
        userAgent: navigator.userAgent,
        location: window.location.href
      };
      
      document.getElementById('music-player').innerHTML = `
        <div style="color:red; padding:1em; background:#ffeeee; border-radius:5px;">
          <h4>⚠️ 音乐播放器故障</h4>
          <p><strong>错误详情:</strong> ${error.message}</p>
          <hr>
          <p><strong>调试信息:</strong></p>
          <pre>${JSON.stringify(debugInfo, null, 2)}</pre>
          <hr>
          <p><strong>请按以下步骤排查:</strong></p>
          <ol>
            <li>访问 <a href="/music.json" target="_blank">music.json</a> 验证文件是否存在</li>
            <li>检查控制台网络请求（按F12）</li>
            <li>确保仓库文件结构正确</li>
          </ol>
        </div>
      `;
    }
  })();
</script>