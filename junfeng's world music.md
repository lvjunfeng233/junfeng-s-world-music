[index.html](https://github.com/user-attachments/files/28389761/index.html)<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>地域音乐 · 表盘</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700&display=swap');
        
        :root {
            --primary: #8b5cf6;
        }
        
        body {
            font-family: 'Noto Sans SC', system-ui, sans-serif;
        }
        
        .glass {
            background: rgba(255, 255, 255, 0.08);
            backdrop-filter: blur(12px);
        }
        
        .song-card {
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        
        .song-card:hover {
            transform: translateY(-8px);
            box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.2);
        }
        
        .nav-active {
            border-bottom: 3px solid #a855f7;
            color: white;
        }
        
        .earth-container {
            height: 520px;
            background: radial-gradient(circle at center, #1e2937 0%, #0f172a 70%);
        }
        
        .playing {
            animation: pulse 2s infinite;
        }
    </style>
</head>
<body class="bg-zinc-950 text-zinc-200 min-h-screen">
    <div class="flex h-screen">
        <!-- 侧边栏 -->
        <div class="w-72 bg-black/90 border-r border-zinc-800 flex flex-col">
            <div class="p-6 border-b border-zinc-800">
                <div class="flex items-center gap-3">
                    <div class="w-10 h-10 bg-gradient-to-br from-purple-500 to-pink-500 rounded-2xl flex items-center justify-center text-white font-bold text-2xl">🌍</div>
                    <div>
                        <h1 class="text-2xl font-bold tracking-tight">地域音乐</h1>
                        <p class="text-xs text-zinc-500">IP定位 · 阿联酋</p>
                    </div>
                </div>
            </div>
            
            <div class="p-4 flex-1">
                <div onclick="switchTab(0)" class="nav-item flex items-center gap-3 px-4 py-3 rounded-2xl hover:bg-white/10 cursor-pointer nav-active" id="nav-0">
                    <i class="fa-solid fa-house w-5"></i>
                    <span class="font-medium">音乐馆</span>
                </div>
                <div onclick="switchTab(1)" class="nav-item flex items-center gap-3 px-4 py-3 rounded-2xl hover:bg-white/10 cursor-pointer mt-1" id="nav-1">
                    <i class="fa-solid fa-heart w-5"></i>
                    <span class="font-medium">我的喜欢</span>
                </div>
                <div onclick="switchTab(2)" class="nav-item flex items-center gap-3 px-4 py-3 rounded-2xl hover:bg-white/10 cursor-pointer mt-1" id="nav-2">
                    <i class="fa-solid fa-globe w-5"></i>
                    <span class="font-medium">地球探索</span>
                </div>
            </div>
            
            <!-- 当前播放 -->
            <div class="p-4 border-t border-zinc-800">
                <div class="glass rounded-3xl p-4">
                    <div class="flex gap-3">
                        <img id="now-cover" src="https://picsum.photos/id/1015/300/300" class="w-16 h-16 rounded-2xl object-cover" alt="">
                        <div class="flex-1 min-w-0">
                            <p id="now-title" class="font-medium truncate">Ya Tabtab</p>
                            <p id="now-artist" class="text-sm text-zinc-400 truncate">Nancy Ajram</p>
                        </div>
                    </div>
                    <div class="mt-4">
                        <audio id="player" class="w-full" controls></audio>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- 主内容 -->
        <div class="flex-1 flex flex-col">
            <!-- 顶部栏 -->
            <div class="h-16 border-b border-zinc-800 bg-black/70 flex items-center px-8 justify-between">
                <div class="flex items-center gap-8">
                    <div onclick="switchTab(0)" class="cursor-pointer text-lg font-semibold flex items-center gap-2" id="tab-0">
                        <span>音乐馆</span>
                    </div>
                    <div onclick="switchTab(1)" class="cursor-pointer text-lg font-semibold flex items-center gap-2 text-zinc-400 hover:text-white" id="tab-1">
                        <span>我的喜欢</span>
                    </div>
                    <div onclick="switchTab(2)" class="cursor-pointer text-lg font-semibold flex items-center gap-2 text-zinc-400 hover:text-white" id="tab-2">
                        <span>地球探索</span>
                    </div>
                </div>
                
                <div class="flex items-center gap-4">
                    <div class="relative">
                        <input id="search-input" 
                               onkeyup="if(event.key==='Enter') searchMusic()"
                               type="text" 
                               placeholder="搜索歌曲、歌手、地区..."
                               class="w-80 bg-zinc-900 border border-zinc-700 rounded-3xl py-2.5 pl-10 text-sm focus:outline-none focus:border-purple-500">
                        <i class="fa-solid fa-magnifying-glass absolute left-4 top-3 text-zinc-500"></i>
                    </div>
                    <div class="w-8 h-8 bg-zinc-800 rounded-2xl flex items-center justify-center cursor-pointer">
                        <i class="fa-solid fa-user"></i>
                    </div>
                </div>
            </div>
            
            <!-- 页面内容 -->
            <!-- 音乐馆 -->
            <div id="page-0" class="flex-1 overflow-auto p-8">
                <div class="mb-8">
                    <h2 class="text-3xl font-bold mb-2">欢迎来到阿联酋音乐馆 <span class="text-amber-400">🇦🇪</span></h2>
                    <p class="text-zinc-400">根据您的IP位置，为您推荐中东风情音乐</p>
                </div>
                
                <!-- 推荐歌单 -->
                <div class="mb-10">
                    <h3 class="text-xl font-semibold mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-list"></i> 精选歌单
                    </h3>
                    <div class="grid grid-cols-5 gap-6">
                        <div onclick="playPlaylist(0)" class="song-card cursor-pointer bg-zinc-900 rounded-3xl overflow-hidden">
                            <img src="https://picsum.photos/id/1015/400/300" class="w-full h-40 object-cover">
                            <div class="p-4">
                                <p class="font-semibold">阿拉伯之夜</p>
                                <p class="text-xs text-zinc-500">中东流行 · 42首</p>
                            </div>
                        </div>
                        <div onclick="playPlaylist(1)" class="song-card cursor-pointer bg-zinc-900 rounded-3xl overflow-hidden">
                            <img src="https://picsum.photos/id/201/400/300" class="w-full h-40 object-cover">
                            <div class="p-4">
                                <p class="font-semibold">迪拜节奏</p>
                                <p class="text-xs text-zinc-500">电子 + 传统</p>
                            </div>
                        </div>
                        <div onclick="playPlaylist(2)" class="song-card cursor-pointer bg-zinc-900 rounded-3xl overflow-hidden">
                            <img src="https://picsum.photos/id/237/400/300" class="w-full h-40 object-cover">
                            <div class="p-4">
                                <p class="font-semibold">沙漠之声</p>
                                <p class="text-xs text-zinc-500">民谣 · 纯享</p>
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- 推荐歌曲 -->
                <div>
                    <h3 class="text-xl font-semibold mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-fire"></i> 今日推荐
                    </h3>
                    <div id="recommend-list" class="grid grid-cols-4 gap-6">
                        <!-- JS填充 -->
                    </div>
                </div>
            </div>
            
            <!-- 我的喜欢 -->
            <div id="page-1" class="flex-1 overflow-auto p-8 hidden">
                <h2 class="text-3xl font-bold mb-8">我的喜欢</h2>
                <div id="favorites-list" class="space-y-3">
                    <!-- JS填充 -->
                </div>
            </div>
            
            <!-- 地球探索 -->
            <div id="page-2" class="flex-1 overflow-auto hidden">
                <div class="p-8">
                    <h2 class="text-3xl font-bold mb-2">点击地球 · 聆听当地</h2>
                    <p class="text-zinc-400 mb-6">探索世界各地的音乐文化</p>
                    
                    <div class="earth-container rounded-3xl overflow-hidden border border-zinc-700 relative" id="map-container">
                        <!-- Leaflet地图将在这里初始化 -->
                    </div>
                    
                    <div class="mt-6 text-center text-sm text-zinc-500">
                        当前位置：<span class="text-emerald-400" id="current-location">阿联酋 · 富查伊拉</span>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- 底部音乐栏 -->
    <div class="fixed bottom-0 left-0 right-0 h-20 bg-zinc-900 border-t border-zinc-700 flex items-center px-6 z-50">
        <div class="flex items-center gap-4 flex-1">
            <img id="bottom-cover" src="https://picsum.photos/id/1015/80/80" class="w-12 h-12 rounded-2xl">
            <div>
                <p id="bottom-title" class="font-medium">Ya Tabtab</p>
                <p id="bottom-artist" class="text-xs text-zinc-400">Nancy Ajram</p>
            </div>
            <button onclick="toggleLike()" class="ml-4 text-xl text-pink-500">
                <i class="fa-solid fa-heart"></i>
            </button>
        </div>
        
        <div class="flex-1 flex justify-center items-center gap-6">
            <button onclick="prevTrack()" class="text-2xl text-zinc-400 hover:text-white">
                <i class="fa-solid fa-backward"></i>
            </button>
            <button onclick="togglePlay()" id="play-btn"
                    class="w-12 h-12 bg-white text-black rounded-2xl flex items-center justify-center text-2xl hover:scale-105 transition">
                <i class="fa-solid fa-play"></i>
            </button>
            <button onclick="nextTrack()" class="text-2xl text-zinc-400 hover:text-white">
                <i class="fa-solid fa-forward"></i>
            </button>
        </div>
        
        <div class="flex-1 flex justify-end items-center gap-3 text-sm">
            <span id="current-time">0:00</span>
            <div class="w-48 h-1 bg-zinc-700 rounded-full relative">
                <div id="progress-bar" class="absolute h-1 bg-purple-500 rounded-full w-1/3"></div>
            </div>
            <span id="duration">3:45</span>
        </div>
    </div>

    <script>
        // 歌曲数据库
        let songs = [
            {
                id: 1,
                title: "Ya Tabtab",
                artist: "Nancy Ajram",
                cover: "https://picsum.photos/id/1015/400/400",
                audio: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3",
                region: "阿联酋"
            },
            {
                id: 2,
                title: "Enta Eih",
                artist: "Nancy Ajram",
                cover: "https://picsum.photos/id/201/400/400",
                audio: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3",
                region: "埃及"
            },
            {
                id: 3,
                title: "Habibi Ya Nour El Ain",
                artist: "Amr Diab",
                cover: "https://picsum.photos/id/237/400/400",
                audio: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3",
                region: "埃及"
            },
            {
                id: 4,
                title: "Desert Rose",
                artist: "Sting ft. Cheb Mami",
                cover: "https://picsum.photos/id/866/400/400",
                audio: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-4.mp3",
                region: "阿尔及利亚"
            },
            {
                id: 5,
                title: "Bint El Shalabiya",
                artist: "Fairuz",
                cover: "https://picsum.photos/id/1005/400/400",
                audio: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-5.mp3",
                region: "黎巴嫩"
            }
        ]
        
        let favorites = []
        let currentTrack = 0
        let isPlaying = false
        let audioPlayer = null
        
        // Tailwind
        function initTailwind() {
            // 已通过script引入
        }
        
        // 渲染推荐歌曲
        function renderRecommendations() {
            const container = document.getElementById('recommend-list')
            container.innerHTML = ''
            
            songs.forEach(song => {
                const div = document.createElement('div')
                div.className = `song-card bg-zinc-900 rounded-3xl overflow-hidden cursor-pointer`
                div.innerHTML = `
                    <div class="relative">
                        <img src="${song.cover}" class="w-full h-52 object-cover">
                        <div onclick="playSong(${song.id}); event.stopImmediatePropagation()" 
                             class="absolute bottom-4 right-4 w-10 h-10 bg-white/90 hover:bg-white rounded-2xl flex items-center justify-center text-xl shadow-lg">
                            <i class="fa-solid fa-play text-black"></i>
                        </div>
                    </div>
                    <div class="p-4">
                        <p class="font-semibold">${song.title}</p>
                        <p class="text-sm text-zinc-400">${song.artist}</p>
                        <p class="text-xs text-emerald-400 mt-1">${song.region}</p>
                    </div>
                `
                container.appendChild(div)
            })
        }
        
        // 渲染我的喜欢
        function renderFavorites() {
            const container = document.getElementById('favorites-list')
            container.innerHTML = ''
            
            if (favorites.length === 0) {
                container.innerHTML = `<div class="text-center py-20 text-zinc-500">还没有喜欢的歌曲～</div>`
                return
            }
            
            favorites.forEach((song, index) => {
                const div = document.createElement('div')
                div.className = `flex items-center gap-4 bg-zinc-900 rounded-3xl p-4 hover:bg-zinc-800 cursor-pointer`
                div.innerHTML = `
                    <img src="${song.cover}" class="w-16 h-16 rounded-2xl object-cover">
                    <div class="flex-1">
                        <p class="font-medium">${song.title}</p>
                        <p class="text-sm text-zinc-400">${song.artist}</p>
                    </div>
                    <button onclick="playSong(${song.id}); event.stopImmediatePropagation()" 
                            class="px-6 py-2 bg-purple-600 hover:bg-purple-700 rounded-3xl text-sm">播放</button>
                    <button onclick="removeFavorite(${index}); event.stopImmediatePropagation()" 
                            class="text-red-400 hover:text-red-500">
                        <i class="fa-solid fa-trash"></i>
                    </button>
                `
                container.appendChild(div)
            })
        }
        
        function playSong(id) {
            const song = songs.find(s => s.id === id)
            if (!song) return
            
            currentTrack = songs.indexOf(song)
            playCurrentTrack()
        }
        
        function playCurrentTrack() {
            const song = songs[currentTrack]
            
            // 更新顶部
            document.getElementById('now-cover').src = song.cover
            document.getElementById('now-title').textContent = song.title
            document.getElementById('now-artist').textContent = song.artist
            
            // 更新底部
            document.getElementById('bottom-cover').src = song.cover
            document.getElementById('bottom-title').textContent = song.title
            document.getElementById('bottom-artist').textContent = song.artist
            
            // 播放音频
            if (!audioPlayer) {
                audioPlayer = document.getElementById('player')
            }
            
            audioPlayer.src = song.audio
            audioPlayer.play()
            isPlaying = true
            updatePlayButton()
        }
        
        function togglePlay() {
            if (!audioPlayer) return
            if (isPlaying) {
                audioPlayer.pause()
            } else {
                audioPlayer.play()
            }
            isPlaying = !isPlaying
            updatePlayButton()
        }
        
        function updatePlayButton() {
            const btn = document.getElementById('play-btn')
            btn.innerHTML = isPlaying ? 
                `<i class="fa-solid fa-pause"></i>` : 
                `<i class="fa-solid fa-play"></i>`
        }
        
        function nextTrack() {
            currentTrack = (currentTrack + 1) % songs.length
            playCurrentTrack()
        }
        
        function prevTrack() {
            currentTrack = (currentTrack - 1 + songs.length) % songs.length
            playCurrentTrack()
        }
        
        function toggleLike() {
            const currentSong = songs[currentTrack]
            if (!favorites.find(f => f.id === currentSong.id)) {
                favorites.push(currentSong)
                renderFavorites()
                alert(`已添加到喜欢：${currentSong.title}`)
            } else {
                alert('已经在喜欢列表中')
            }
        }
        
        function removeFavorite(index) {
            favorites.splice(index, 1)
            renderFavorites()
        }
        
        // 标签切换
        function switchTab(tab) {
            for (let i = 0; i < 3; i++) {
                document.getElementById(`page-${i}`).classList.add('hidden')
                document.getElementById(`tab-${i}`).classList.remove('text-white')
                document.getElementById(`tab-${i}`).classList.add('text-zinc-400')
                document.getElementById(`nav-${i}`).classList.remove('nav-active')
            }
            
            document.getElementById(`page-${tab}`).classList.remove('hidden')
            document.getElementById(`tab-${tab}`).classList.add('text-white')
            document.getElementById(`nav-${tab}`).classList.add('nav-active')
        }
        
        // 模拟搜索
        function searchMusic() {
            const query = document.getElementById('search-input').value.toLowerCase().trim()
            if (!query) return
            
            const filtered = songs.filter(song => 
                song.title.toLowerCase().includes(query) || 
                song.artist.toLowerCase().includes(query)
            )
            
            if (filtered.length > 0) {
                alert(`找到 ${filtered.length} 首相关歌曲：\n` + filtered.map(s => s.title).join('\n'))
                // 可以切换到音乐馆并渲染过滤结果
            } else {
                alert('没有找到相关歌曲')
            }
        }
        
        // 播放歌单
        function playPlaylist(n) {
            alert(`正在播放歌单 #${n+1}...`)
            // 随机播放一首
            currentTrack = Math.floor(Math.random() * songs.length)
            playCurrentTrack()
        }
        
        // 初始化Leaflet地图
        function initMap() {
            const mapContainer = document.getElementById('map-container')
            mapContainer.innerHTML = `<div id="leaflet-map" style="height: 100%; width: 100%;"></div>`
            
            const map = L.map('leaflet-map', {
                center: [25.2, 56.3], // 富查伊拉附近
                zoom: 3,
                zoomControl: true
            })
            
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; OpenStreetMap',
                className: 'map-tiles'
            }).addTo(map)
            
            // 添加一些标记点
            const regions = [
                {coords: [25.2, 56.3], name: "阿联酋", songIndex: 0},
                {coords: [30.0, 31.2], name: "埃及", songIndex: 1},
                {coords: [33.9, 35.5], name: "黎巴嫩", songIndex: 4},
                {coords: [36.7, 3.2], name: "阿尔及利亚", songIndex: 3}
            ]
            
            regions.forEach(region => {
                const marker = L.marker(region.coords).addTo(map)
                marker.bindPopup(`
                    <b>${region.name}</b><br>
                    <button onclick="playLocalMusic(${region.songIndex});" 
                            class="mt-2 px-4 py-1 bg-purple-600 text-white rounded-2xl text-xs">播放当地音乐</button>
                `)
            })
            
            // 点击地图任意位置提示
            map.on('click', function(e) {
                const lat = e.latlng.lat
                const lng = e.latlng.lng
                
                // 简单区域判断
                let location = "未知地区"
                if (lat > 20 && lat < 35 && lng > 40 && lng < 65) {
                    location = "中东"
                } else if (lat > 35 && lat < 50 && lng > 0 && lng < 30) {
                    location = "欧洲"
                }
                
                alert(`您点击了经纬度 (${lat.toFixed(1)}, \( {lng.toFixed(1)})\n\n正在为您播放 \){location}风格的音乐...`)
                currentTrack = Math.floor(Math.random() * songs.length)
                playCurrentTrack()
            })
        }
        
        function playLocalMusic(index) {
            currentTrack = index
            playCurrentTrack()
        }
        
        // 初始化
        function initialize() {
            renderRecommendations()
            renderFavorites()
            switchTab(0)
            initMap()
            
            // 默认播放第一首
            setTimeout(() => {
                currentTrack = 0
                playCurrentTrack()
            }, 800)
            
            // 音频事件
            setTimeout(() => {
                audioPlayer = document.getElementById('player')
                audioPlayer.addEventListener('ended', () => {
                    nextTrack()
                })
            }, 1000)
            
            console.log('%c地域音乐表盘已加载完成 🌍', 'color:#a855f7; font-size:14px')
        }
        
        window.onload = initialize
    </script>
</body>
</html>
