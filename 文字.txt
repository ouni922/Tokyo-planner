<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>日系極簡行程規劃 App</title>
    <!-- 導入 Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 導入 Vue 3 CDN -->
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <!-- 導入 Lucide Icons CDN (用於美觀的 Icon) -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <!-- Tailwind Config: 定義日系極簡和冰雪藍棕色調 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'snow-light': '#F8F9FA',      // 冰雪風格淺色背景
                        'snow-blue': '#E9ECEF',       // 冰雪風格次要藍
                        'primary-blue': '#1F2937',    // 深藍/深棕取代 (日系內斂深色)
                        'accent-brown': '#7C3A01',    // 溫暖的棕色重點
                        'subtle-gray': '#D1D5DB',     // 極簡風格的線條灰
                    },
                    fontFamily: {
                        sans: ['Inter', 'Noto Sans TC', 'sans-serif'], // 確保支援中文
                    }
                }
            }
        }
    </script>
    <style>
        /* 讓 App 內容區域能填滿整個手機螢幕 */
        body, #app {
            height: 100vh;
            margin: 0;
            display: flex;
            flex-direction: column;
            font-family: 'Inter', 'Noto Sans TC', sans-serif;
            background-color: #F8F9FA; /* 冰雪淺色背景 */
        }
        /* 固定地圖高度，讓它不會佔滿手機螢幕 */
        #map {
            width: 100%;
            height: 350px; 
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
        }
        /* 調整輸入框樣式，確保極簡美感 */
        input:focus {
            border-color: #1F2937 !important; /* 深藍色邊框 */
            box-shadow: none !important;
        }
        /* 隱藏地圖載入錯誤提示 */
        .gm-err-container {
            display: none !important;
        }
    </style>
</head>
<body>

    <div id="app" class="max-w-md mx-auto w-full overflow-y-auto">

        <!-- 頁首 Banner (滿版, 250px 高, Placeholder Image) -->
        <header :style="{ backgroundImage: 'url(' + bannerImageUrl + ')' }"
                class="w-full bg-cover bg-center h-[250px] relative shadow-lg">
            <div class="absolute inset-0 bg-primary-blue bg-opacity-30 flex items-end p-4 rounded-b-lg">
                <h1 class="text-3xl font-bold text-white tracking-wider drop-shadow-md">
                    旅行日誌
                </h1>
            </div>
        </header>

        <!-- 主要內容區域 -->
        <main class="p-4 space-y-6">

            <!-- 地圖顯示區域 -->
            <div class="bg-white p-4 rounded-xl shadow-md border border-snow-blue">
                <h2 class="text-lg font-semibold mb-3 text-primary-blue flex items-center">
                    <i data-lucide="map-pin" class="w-5 h-5 mr-2"></i>
                    地圖概覽
                </h2>
                <!-- Google Maps 容器 -->
                <div id="map">
                    <!-- 地圖載入中提示 -->
                    <div v-if="!isMapReady" class="flex items-center justify-center h-full text-subtle-gray">
                        正在載入地圖... (請檢查 API 金鑰)
                    </div>
                </div>
            </div>

            <!-- 新增行程點位區塊 -->
            <div class="bg-white p-4 rounded-xl shadow-md border border-snow-blue">
                <h2 class="text-lg font-semibold mb-4 text-primary-blue flex items-center">
                    <i data-lucide="plus-circle" class="w-5 h-5 mr-2"></i>
                    新增地點
                </h2>
                <div class="flex space-x-2">
                    <input type="text" v-model="newLocationQuery"
                           placeholder="輸入地點名稱或地址..."
                           class="flex-grow p-3 border border-subtle-gray rounded-lg focus:ring-1 focus:ring-primary-blue transition duration-150 text-sm"
                           @keyup.enter="addLocation">
                    <button @click="addLocation"
                            :disabled="!newLocationQuery.trim()"
                            class="p-3 bg-accent-brown text-white rounded-lg shadow-md hover:bg-opacity-90 disabled:bg-gray-400 transition duration-150 flex items-center justify-center">
                        <i data-lucide="send" class="w-5 h-5"></i>
                    </button>
                </div>
                <!-- 錯誤訊息 -->
                <p v-if="error" class="mt-2 text-sm text-red-600">{{ error }}</p>
            </div>

            <!-- 行程清單區塊 -->
            <div class="bg-white p-4 rounded-xl shadow-md border border-snow-blue">
                <h2 class="text-lg font-semibold mb-4 text-primary-blue flex items-center">
                    <i data-lucide="list-ordered" class="w-5 h-5 mr-2"></i>
                    我的行程點位 (共 {{ locations.length }} 點)
                </h2>
                <div v-if="locations.length === 0" class="text-center py-6 text-subtle-gray italic">
                    <p>尚未新增任何地點。開始您的極簡旅程吧！</p>
                </div>

                <ul class="space-y-3">
                    <li v-for="(location, index) in locations" :key="location.id"
                        class="flex items-center justify-between p-3 bg-snow-light border-l-4 border-accent-brown rounded-lg shadow-sm">
                        
                        <div class="flex items-center min-w-0 mr-4">
                             <span class="text-xl font-bold text-accent-brown mr-3">{{ index + 1 }}</span>
                             <span class="truncate text-gray-800 font-medium">{{ location.name }}</span>
                        </div>
                       
                        <div class="flex space-x-2 flex-shrink-0">
                            <!-- 導航按鈕：開啟 Google Maps App/網頁進行導航 -->
                            <a :href="location.navigationUrl" target="_blank"
                               class="p-2 bg-primary-blue text-white rounded-full hover:bg-opacity-90 transition duration-150"
                               title="開始導航">
                                <i data-lucide="navigation" class="w-4 h-4"></i>
                            </a>
                            <!-- 刪除按鈕 -->
                            <button @click="removeLocation(location.id)"
                                    class="p-2 bg-gray-300 text-gray-800 rounded-full hover:bg-gray-400 transition duration-150"
                                    title="移除地點">
                                <i data-lucide="trash-2" class="w-4 h-4"></i>
                            </button>
                        </div>
                    </li>
                </ul>
            </div>
        </main>

        <!-- 版權資訊 (日系極簡風格的底部) -->
        <footer class="w-full text-center py-4 text-xs text-subtle-gray bg-white mt-auto border-t border-snow-blue">
            © 2025 極簡旅程。所有權利保留。
        </footer>

    </div>

    <!-- Google Maps 腳本 -->
    <!-- !!! 請將 YOUR_GOOGLE_MAPS_API_KEY 替換成您的 API 金鑰 !!! -->
    <script async defer
        src="https://maps.googleapis.com/maps/api/js?key=YOUR_GOOGLE_MAPS_API_KEY&libraries=places&callback=initMap">
    </script>


    <!-- Vue 應用程式腳本 -->
    <script>
        // ----------------------------------------------------
        // Helper 函式：用於 Base64 編碼/解碼 (如果需要)
        const base64Encode = (str) => btoa(unescape(encodeURIComponent(str)));
        const base64Decode = (str) => decodeURIComponent(escape(atob(str)));
        // ----------------------------------------------------

        const { createApp, ref, onMounted, computed, nextTick } = Vue

        // 初始化 Google Map 的全域函式 (由 Google Maps API 呼叫)
        window.initMap = () => {
            const app = createApp({
                setup() {
                    // Vue 狀態
                    const locations = ref([]);
                    const newLocationQuery = ref('');
                    const map = ref(null);
                    const mapCenter = ref({ lat: 25.0330, lng: 121.5654 }); // 預設台北 101
                    const isMapReady = ref(false);
                    const error = ref('');
                    let nextId = 1;

                    // 地圖物件和服務
                    let geocoder = null;
                    let markers = [];

                    // Banner 圖片 (請替換為您的圖片 URL)
                    const bannerImageUrl = 'https://placehold.co/1920x400/9BB8D3/000000?text=XXX+%E6%A9%AB%E5%B9%85%E5%9C%96%E7%89%87%E4%BD%8D%E7%BD%AE';

                    /**
                     * 初始化 Google 地圖
                     */
                    const initializeMap = () => {
                        const mapInstance = new google.maps.Map(document.getElementById("map"), {
                            center: mapCenter.value,
                            zoom: 12,
                            disableDefaultUI: true, // 保持極簡風格
                            zoomControl: true,
                        });

                        map.value = mapInstance;
                        geocoder = new google.maps.Geocoder();
                        isMapReady.value = true;
                        console.log("Google Maps 初始化成功");
                        
                        // 重新繪製 Lucide Icons
                        nextTick(() => {
                            lucide.createIcons();
                        });

                        // 載入 localStorage 中已儲存的點位
                        loadLocations();
                    };

                    /**
                     * 將地點名稱轉換為地理座標，並新增到行程中
                     */
                    const addLocation = () => {
                        error.value = '';
                        const query = newLocationQuery.value.trim();
                        if (!query || !geocoder) {
                            error.value = '請輸入有效地點。';
                            return;
                        }

                        geocoder.geocode({ address: query }, (results, status) => {
                            if (status === 'OK' && results[0]) {
                                const place = results[0];
                                const latLng = {
                                    lat: place.geometry.location.lat(),
                                    lng: place.geometry.location.lng(),
                                };
                                const newLocation = {
                                    id: nextId++,
                                    name: place.formatted_address, // 使用地圖提供的完整地址
                                    lat: latLng.lat,
                                    lng: latLng.lng,
                                    // 產生 Google Maps 導航連結
                                    navigationUrl: `https://www.google.com/maps/dir/?api=1&destination=${latLng.lat},${latLng.lng}&travelmode=driving`,
                                };
                                locations.value.push(newLocation);
                                saveLocations();
                                updateMapMarkers();
                                newLocationQuery.value = '';
                                
                                // 重新繪製 Lucide Icons
                                nextTick(() => {
                                    lucide.createIcons();
                                });
                            } else {
                                console.error('Geocode 失敗:', status);
                                error.value = '找不到該地點，請嘗試更明確的名稱。';
                            }
                        });
                    };

                    /**
                     * 從行程中移除地點
                     */
                    const removeLocation = (id) => {
                        const index = locations.value.findIndex(loc => loc.id === id);
                        if (index !== -1) {
                            locations.value.splice(index, 1);
                            saveLocations();
                            updateMapMarkers();
                        }
                    };

                    /**
                     * 更新地圖上的標記點
                     */
                    const updateMapMarkers = () => {
                        // 清除現有的標記
                        markers.forEach(marker => marker.setMap(null));
                        markers = [];

                        if (locations.value.length === 0) {
                            // 如果沒有地點，將地圖中心重置回預設
                            map.value.setCenter(mapCenter.value);
                            map.value.setZoom(12);
                            return;
                        }

                        let bounds = new google.maps.LatLngBounds();

                        locations.value.forEach((location, index) => {
                            const position = { lat: location.lat, lng: location.lng };
                            const marker = new google.maps.Marker({
                                position: position,
                                map: map.value,
                                title: location.name,
                                label: {
                                    text: String(index + 1),
                                    color: 'white',
                                    fontWeight: 'bold',
                                },
                                // 使用客製化 Icon 營造極簡風格
                                icon: {
                                    path: google.maps.SymbolPath.CIRCLE,
                                    fillColor: '#7C3A01', // 棕色 accent
                                    fillOpacity: 0.9,
                                    scale: 10,
                                    strokeColor: 'white',
                                    strokeWeight: 2,
                                }
                            });
                            markers.push(marker);
                            bounds.extend(position);
                        });

                        // 調整地圖以包含所有標記點
                        if (!bounds.isEmpty()) {
                            map.value.fitBounds(bounds);
                            // 修正單一標記點時縮放過大的問題
                            if (locations.value.length === 1) {
                                map.value.setZoom(15);
                            }
                        }
                    };

                    /**
                     * 儲存地點到瀏覽器 localStorage
                     */
                    const saveLocations = () => {
                        try {
                            const locationsData = JSON.stringify(locations.value);
                            localStorage.setItem('tripLocations', base64Encode(locationsData));
                            localStorage.setItem('nextId', nextId);
                            console.log('行程已儲存。');
                        } catch (e) {
                            console.error('儲存行程失敗:', e);
                        }
                    };

                    /**
                     * 從瀏覽器 localStorage 載入地點
                     */
                    const loadLocations = () => {
                        try {
                            const locationsData = localStorage.getItem('tripLocations');
                            const savedNextId = localStorage.getItem('nextId');

                            if (locationsData) {
                                const decodedData = base64Decode(locationsData);
                                locations.value = JSON.parse(decodedData);
                                nextId = savedNextId ? parseInt(savedNextId) : locations.value.length + 1;
                                nextTick(updateMapMarkers); // 確保在 DOM 更新後繪製標記
                            }
                            console.log('行程已載入。');
                        } catch (e) {
                            console.error('載入行程失敗，使用預設值。', e);
                            localStorage.removeItem('tripLocations'); // 清除錯誤資料
                            locations.value = [];
                        }
                    };

                    // 確保在地圖初始化後執行
                    onMounted(() => {
                        // 在此處不直接呼叫 initializeMap，而是依賴 Google Maps script 載入後回調 `window.initMap`
                        // 這是 Google Maps API 的標準做法
                    });

                    return {
                        // 狀態
                        locations,
                        newLocationQuery,
                        isMapReady,
                        error,
                        bannerImageUrl,
                        // 方法
                        addLocation,
                        removeLocation,
                        // 供 Google Maps API 呼叫的初始化方法
                        initMap: initializeMap 
                    }
                }
            });

            app.mount('#app');
        };
    </script>

</body>
</html>

