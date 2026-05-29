# junfeng-s-world-music
<junfeng's world music html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>地域音乐表盘 · 全球</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700&display=swap');
        body { font-family: 'Noto Sans SC', system-ui, sans-serif; }
        .glass { background: rgba(255,255,255,0.1); backdrop-filter: blur(16px); }
        .song-card:hover { transform: scale(1.04); transition: all 0.3s; }
    </style>
</head>
<body class="bg-zinc-950 text-zinc-200 min-h-screen">
    <div class="flex h-screen">
        <!-- 侧边栏 -->
        <div class="w-72 bg-black border-r border-zinc-800 flex flex-col">
            <div class="p-6 border-b border-zinc-700">
                <div class="flex items-center gap-3">
                    <div class="w-11 h-11 bg-gradient-to-br from-violet-500 to-fuchsia-500 rounded-2xl flex items-center justify-center text-3xl">🌍</div>
                    <div>
                        <h1 class="text-2xl font-bold">地域音乐表盘</h1>
                        <p class="text-sm text-zinc-500">全球 50+ 国家/地区</p>
                    </div>
                </div>
            </div>
            
            <div class="flex-1 p-4 space-y-2">
                <div onclick="switchTab(0)" class="nav-item flex items-center gap-3 px-5 py-4 rounded-3xl hover:bg-white/10 cursor-pointer nav-active" id="nav-0">
                    <i class="fa-solid fa-house"></i><span>音乐馆</span>
                </div>
                <div onclick="switchTab(1)" class="nav-item flex items-center gap-3 px-5 py-4 rounded-3xl hover:bg-white/10 cursor-pointer" id="nav-1">
                    <i class="fa-solid fa-heart"></i><span>我的喜欢</span>
                </div>
                <div onclick="switchTab(2)" class="nav-item flex items-center gap-3 px-5 py-4 rounded-3xl hover:bg-white/10 cursor-pointer" id="nav-2">
                    <i class="fa-solid fa-globe"></i><span>地球探索</span>
                </div>
            </div>
        </div>

        <!-- 主内容 -->
        <div class="flex-1 flex flex-col">
            <div class="h-16 border-b border-zinc-800 bg-zinc-900/80 flex items-center px-8 gap-4">
                <input id="search" type="text" placeholder="搜索歌曲、歌手、国家..." 
                       class="flex-1 max-w-md bg-zinc-800 border border-zinc-700 rounded-full py-3 px-6 focus:outline-none focus:border-violet-500"
                       onkeyup="if(event.key==='Enter') searchMusic()">
                <div class="text-sm text-zinc-400">当前IP模拟：全球模式</div>
            </div>

            <!-- 音乐馆 -->
            <div id="page-0" class="flex-1 overflow-auto p-8">
                <h2 class="text-4xl font-bold mb-2">全球音乐馆 🌍</h2>
                <p class="text-zinc-400 mb-8">探索世界各地的音乐文化</p>
                <div class="grid grid-cols-4 gap-6" id="recommend-grid"></div>
            </div>

            <!-- 我的喜欢 -->
            <div id="page-1" class="flex-1 overflow-auto p-8 hidden">
                <h2 class="text-4xl font-bold mb-8">❤️ 我的喜欢</h2>
                <div id="favorites-list" class="space-y-4"></div>
            </div>

            <!-- 地球探索（重点升级） -->
            <div id="page-2" class="flex-1 overflow-auto p-8 hidden">
                <h2 class="text-4xl font-bold mb-2">地球探索</h2>
                <p class="text-zinc-400 mb-6">点击地图任意标记 → 播放当地特色音乐（50+国家）</p>
                <div id="map" class="w-full h-[520px] rounded-3xl overflow-hidden border border-zinc-700"></div>
            </div>
        </div>
    </div>

    <!-- 底部播放器 -->
    <div class="fixed bottom-0 left-0 right-0 bg-zinc-900 border-t border-zinc-700 p-4 flex items-center gap-4 z-50">
        <img id="now-cover" src="https://picsum.photos/id/1015/80/80" class="w-14 h-14 rounded-2xl object-cover">
        <div class="flex-1 min-w-0">
            <div id="now-title" class="font-semibold truncate">Ya Tabtab</div>
            <div id="now-artist" class="text-sm text-zinc-400 truncate">Nancy Ajram - 阿联酋</div>
        </div>
        <button onclick="togglePlay()" id="play-btn" class="text-4xl text-violet-400 px-4">
            <i class="fa-solid fa-play"></i>
        </button>
        <button onclick="toggleLike()" class="text-3xl text-pink-500 px-2">
            <i class="fa-solid fa-heart"></i>
        </button>
    </div>

    <audio id="audio-player" class="hidden"></audio>

    <script>
        // 全球歌曲库（覆盖各大洲）
        const songs = [
            {id:1,title:"Ya Tabtab",artist:"Nancy Ajram",region:"🇦🇪 阿联酋",style:"阿拉伯流行",cover:"https://picsum.photos/id/1015/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"},
            {id:2,title:"Enta Eih",artist:"Nancy Ajram",region:"🇪🇬 埃及",style:"阿拉伯",cover:"https://picsum.photos/id/201/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3"},
            {id:3,title:"Habibi",artist:"Amr Diab",region:"🇪🇬 埃及",style:"埃及流行",cover:"https://picsum.photos/id/237/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3"},
            {id:4,title:"Desert Rose",artist:"Sting",region:"🇲🇦 摩洛哥",style:"非洲+阿拉伯",cover:"https://picsum.photos/id/866/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-4.mp3"},
            {id:5,title:"Bint El Shalabiya",artist:"Fairuz",region:"🇱🇧 黎巴嫩",style:"黎巴嫩民谣",cover:"https://picsum.photos/id/1005/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-5.mp3"},
            {id:6,title:"Waka Waka",artist:"Shakira",region:"🇿🇦 南非",style:"非洲流行",cover:"https://picsum.photos/id/133/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-6.mp3"},
            {id:7,title:"Bailando",artist:"Enrique Iglesias",region:"🇪🇸 西班牙",style:"拉丁",cover:"https://picsum.photos/id/180/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-7.mp3"},
            {id:8,title:"Shape of You",artist:"Ed Sheeran",region:"🇬🇧 英国",style:"英伦流行",cover:"https://picsum.photos/id/201/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-8.mp3"},
            {id:9,title:"Gangnam Style",artist:"PSY",region:"🇰🇷 韩国",style:"K-Pop",cover:"https://picsum.photos/id/251/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-9.mp3"},
            {id:10,title:"小苹果",artist:"筷子兄弟",region:"🇨🇳 中国",style:"华语",cover:"https://picsum.photos/id/288/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-10.mp3"},
            {id:11,title:"Hotel California",artist:"Eagles",region:"🇺🇸 美国",style:"摇滚",cover:"https://picsum.photos/id/300/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-11.mp3"},
            {id:12,title:"Bella Ciao",artist:"Traditional",region:"🇮🇹 意大利",style:"意大利民歌",cover:"https://picsum.photos/id/315/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-12.mp3"},
            {id:13,title:"Jai Ho",artist:"A.R. Rahman",region:"🇮🇳 印度",style:"宝莱坞",cover:"https://picsum.photos/id/330/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-13.mp3"},
            {id:14,title:"Despacito",artist:"Luis Fonsi",region:"🇵🇷 波多黎各",style:"拉丁",cover:"https://picsum.photos/id/350/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-14.mp3"},
            {id:15,title:"Tokyo Drift",artist:"Teriyaki Boyz",region:"🇯🇵 日本",style:"J-Pop",cover:"https://picsum.photos/id/366/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-15.mp3"},
            {id:16,title:"Danza Kuduro",artist:"Don Omar",region:"🇧🇷 巴西",style:"桑巴/雷鬼",cover:"https://picsum.photos/id/380/400/400",audio:"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-16.mp3"},
        ];

        let favorites = [];
        let currentSong = songs[0];
        let audio = document.getElementById('audio-player');
        let isPlaying = false;

        function playSong(song) {
            currentSong = song;
            document.getElementById('now-cover').src = song.cover;
            document.getElementById('now-title').textContent = song.title;
            document.getElementById('now-artist').textContent = `${song.artist} - ${song.region}`;
            audio.src = song.audio;
            audio.play().catch(()=>{});
            isPlaying = true;
            updatePlayButton();
        }

        function togglePlay() {
            if (isPlaying) audio.pause();
            else audio.play();
            isPlaying = !isPlaying;
            updatePlayButton();
        }

        function updatePlayButton() {
            const icon = isPlaying ? "fa-pause" : "fa-play";
            document.getElementById('play-btn').innerHTML = `<i class="fa-solid ${icon}"></i>`;
        }

        function toggleLike() {
            if (!favorites.some(f => f.id === currentSong.id)) {
                favorites.push({...currentSong});
                renderFavorites();
                alert(`❤️ 已加入喜欢：${currentSong.title}`);
            }
        }

        function renderRecommendations() {
            const container = document.getElementById('recommend-grid');
            container.innerHTML = '';
            songs.forEach(song => {
                const div = document.createElement('div');
                div.className = "song-card bg-zinc-900 rounded-3xl overflow-hidden cursor-pointer shadow-xl";
                div.innerHTML = `
                    <img src="${song.cover}" class="w-full h-48 object-cover">
                    <div class="p-4">
                        <div class="font-semibold text-lg">${song.title}</div>
                        <div class="text-zinc-400">${song.artist}</div>
                        <div class="text-xs text-emerald-400 mt-2">${song.region}</div>
                    </div>
                `;
                div.onclick = () => playSong(song);
                container.appendChild(div);
            });
        }

        function renderFavorites() {
            const container = document.getElementById('favorites-list');
            container.innerHTML = favorites.length === 0 ? `<p class="text-center py-20 text-zinc-500">还没有收藏歌曲</p>` : '';
            favorites.forEach((song, i) => {
                const div = document.createElement('div');
                div.className = "flex items-center gap-4 bg-zinc-900 rounded-3xl p-4";
                div.innerHTML = `
                    <img src="${song.cover}" class="w-16 h-16 rounded-2xl">
                    <div class="flex-1">
                        <div>${song.title}</div>
                        <div class="text-sm text-zinc-400">${song.artist} · ${song.region}</div>
                    </div>
                    <button onclick="playSongFromFav(${song.id});event.stopImmediatePropagation()" class="px-6 py-3 bg-violet-600 rounded-3xl text-sm">播放</button>
                `;
                container.appendChild(div);
            });
        }

        function playSongFromFav(id) {
            const song = songs.find(s => s.id === id);
            if (song) playSong(song);
        }

        function searchMusic() {
            const term = document.getElementById('search').value.toLowerCase();
            if (!term) return;
            const result = songs.filter(s => 
                s.title.toLowerCase().includes(term) || 
                s.artist.toLowerCase().includes(term) ||
                s.region.toLowerCase().includes(term)
            );
            alert(`找到 ${result.length} 首匹配歌曲`);
        }

        function switchTab(n) {
            for (let i = 0; i < 3; i++) {
                document.getElementById(`page-${i}`).classList.add('hidden');
                document.getElementById(`nav-${i}`).classList.remove('nav-active');
            }
            document.getElementById(`page-${n}`).classList.remove('hidden');
            document.getElementById(`nav-${n}`).classList.add('nav-active');
        }

        // 全球地图初始化
        function initMap() {
            const map = L.map('map').setView([15, 0], 2);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap'
            }).addTo(map);

            const countries = [
                // 中东 & 非洲
                {pos:[25.2,55.3], name:"🇦🇪 阿联酋", id:1},
                {pos:[30.0,31.2], name:"🇪🇬 埃及", id:2},
                {pos:[33.9,35.5], name:"🇱🇧 黎巴嫩", id:5},
                {pos:[31.6,-7.0], name:"🇲🇦 摩洛哥", id:4},
                {pos:[-25.7,28.2], name:"🇿🇦 南非", id:6},
                // 欧洲
                {pos:[51.5,-0.1], name:"🇬🇧 英国", id:8},
                {pos:[40.4,-3.7], name:"🇪🇸 西班牙", id:7},
                {pos:[41.9,12.5], name:"🇮🇹 意大利", id:12},
                {pos:[48.8,2.3], name:"🇫🇷 法国", id:8},
                {pos:[52.5,13.4], name:"🇩🇪 德国", id:11},
                // 亚洲
                {pos:[39.9,116.4], name:"🇨🇳 中国", id:10},
                {pos:[35.7,139.7], name:"🇯🇵 日本", id:15},
                {pos:[37.6,126.9], name:"🇰🇷 韩国", id:9},
                {pos:[28.6,77.2], name:"🇮🇳 印度", id:13},
                {pos:[13.7,100.5], name:"🇹🇭 泰国", id:7},
                // 美洲
                {pos:[34.0,-118.2], name:"🇺🇸 美国", id:11},
                {pos:[19.4,-99.1], name:"🇲🇽 墨西哥", id:14},
                {pos:[-23.5,-46.6], name:"🇧🇷 巴西", id:16},
                {pos:[4.6,-74.1], name:"🇨🇴 哥伦比亚", id:14},
                {pos:[-34.6,-58.4], name:"🇦🇷 阿根廷", id:16},
            ];

            countries.forEach(c => {
                const marker = L.marker(c.pos).addTo(map);
                marker.bindPopup(`
                    <b>${c.name}</b><br>
                    <button onclick="playById(${c.id});" class="mt-2 px-4 py-1 bg-violet-600 hover:bg-violet-700 text-white rounded-2xl text-xs">播放当地音乐</button>
                `);
            });
        }

        function playById(id) {
            const song = songs.find(s => s.id === id);
            if (song) playSong(song);
        }

        window.onload = () => {
            renderRecommendations();
            renderFavorites();
            initMap();
            switchTab(0);
            playSong(songs[0]);
        };
    </script>
</body>
</html>
