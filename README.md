<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🇯🇵 大阪/京都/奈良 - 旅遊規劃 App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <style>
        /* 保持 App 視覺一致性 */
        .app-container {
            overflow: hidden;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
    </style>
</head>
<body class="bg-gray-50">

<div id="app" class="app-container max-w-lg mx-auto shadow-2xl bg-white">
    <header class="p-4 border-b border-gray-100 shadow-sm bg-white sticky top-0 z-10">
        <h1 class="text-xl font-bold text-gray-800 flex items-center">
            <svg class="w-6 h-6 mr-2 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
            🇯🇵 關西旅遊助手
        </h1>
        <p class="text-sm text-gray-500 mt-1">大阪 | 京都 | 奈良</p>
    </header>

    <main class="flex-grow overflow-y-auto p-4 pb-20">
        <div v-if="currentView === 'planner'">
            <TripPlanner />
        </div>
        <div v-else-if="currentView === 'splitter'">
            <ExpenseSplitter />
        </div>
        <div v-else-if="currentView === 'map'">
            <Map />
        </div>
    </main>

    <footer class="fixed bottom-0 left-0 right-0 max-w-lg mx-auto bg-white border-t border-gray-100 shadow-xl flex justify-around items-center p-2 z-20">
        <button 
            @click="currentView = 'planner'"
            :class="['flex flex-col items-center p-2 rounded-lg transition-all', currentView === 'planner' ? 'text-blue-600 bg-blue-50' : 'text-gray-500 hover:text-blue-500']"
        >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
            <span class="text-xs mt-1 font-medium">行程</span>
        </button>

        <button 
            @click="currentView = 'splitter'"
            :class="['flex flex-col items-center p-2 rounded-lg transition-all', currentView === 'splitter' ? 'text-blue-600 bg-blue-50' : 'text-gray-500 hover:text-blue-500']"
        >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
            <span class="text-xs mt-1 font-medium">分帳</span>
        </button>

        <button 
            @click="currentView = 'map'"
            :class="['flex flex-col items-center p-2 rounded-lg transition-all', currentView === 'map' ? 'text-blue-600 bg-blue-50' : 'text-gray-500 hover:text-blue-500']"
        >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.828 0l-4.243-4.243m11.314 0A9 9 0 1111 2.828l.004-.004L21 12l-3.343 3.343z"></path></svg>
            <span class="text-xs mt-1 font-medium">地圖</span>
        </button>
    </footer>

</div>

<script>
    const { createApp, ref, reactive, computed } = Vue

    // --- 1. 行程規劃組件 (TripPlanner) ---
    const TripPlanner = {
        template: `
            <div class="space-y-4">
                <h2 class="text-2xl font-semibold text-gray-800">🗓 每日行程表</h2>
                <div class="flex space-x-2 overflow-x-auto pb-2">
                    <button v-for="day in days" :key="day"
                        @click="selectedDay = day"
                        :class="['px-4 py-2 text-sm font-medium rounded-full transition-colors whitespace-nowrap', 
                                 selectedDay === day ? 'bg-blue-600 text-white shadow-lg' : 'bg-gray-200 text-gray-700 hover:bg-blue-100']">
                        {{ day }}
                    </button>
                </div>

                <div v-for="(schedule, index) in filteredSchedule" :key="index"
                     class="bg-white p-4 border border-gray-100 rounded-xl shadow-md transition-all hover:shadow-lg">
                    <div class="flex justify-between items-start">
                        <div class="flex items-center space-x-3">
                            <span class="text-xl font-bold text-blue-500">{{ schedule.time }}</span>
                            <div>
                                <a v-if="schedule.mapUrl" :href="schedule.mapUrl" target="_blank" class="font-semibold text-lg text-blue-600 hover:text-blue-800 transition-colors flex items-center">
                                    {{ schedule.location }}
                                    <svg class="w-4 h-4 ml-1 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14"></path></svg>
                                </a>
                                <h3 v-else class="font-semibold text-lg text-gray-800">{{ schedule.location }}</h3>
                                
                                <p class="text-sm text-gray-500">{{ schedule.activity }}</p>
                            </div>
                        </div>
                        <button @click="deleteItem(index)" class="text-red-500 hover:text-red-700 p-1 rounded-full bg-red-50/50">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                        </button>
                    </div>
                </div>

                <button class="w-full bg-blue-500 text-white py-3 rounded-xl shadow-lg hover:bg-blue-600 transition-colors mt-4">
                    + 新增行程 (編輯功能待擴展)
                </button>
            </div>
        `,
        setup() {
            // --------------------------------------------------------
            // 🎯 已更新的行程數據 (根據您的輸入) 
            // --------------------------------------------------------
            const selectedDay = ref('Day 1 - 大阪市區精華')
            const days = ['Day 1 - 大阪市區精華', 'Day 2 - 大阪環球影城', 'Day 3 - 奈良一日遊', 'Day 4 - 京都一日深度遊', 'Day 5 - 大阪收尾 + Outlet']
            
            const scheduleData = reactive([
                // --- Day 1 | 大阪市區精華 ---
                { day: 'Day 1 - 大阪市區精華', time: '早餐', location: 'パンの小屋 / 喫茶田園', activity: '現烤麵包/昭和風咖啡', mapUrl: '' },
                { day: 'Day 1 - 大阪市區精華', time: '08:30', location: '天下茶屋 → 大阪城', activity: '搭乘電車 (約35分/¥280)', mapUrl: 'https://maps.google.com/?q=大阪城' },
                { day: 'Day 1 - 大阪市區精華', time: '09:10', location: '大阪城 & 天守閣', activity: '參觀 (09:10-10:40)', mapUrl: '' },
                { day: 'Day 1 - 大阪市區精華', time: '11:10', location: '大阪城 → 黑門市場', activity: '搭乘電車 (約25分/¥230)', mapUrl: 'https://maps.google.com/?q=黑門市場' },
                { day: 'Day 1 - 大阪市區精華', time: '11:30', location: '黑門市場', activity: '午餐 (11:30-12:40)', mapUrl: '' },
                { day: 'Day 1 - 大阪市區精華', time: '12:50', location: '日本橋筋商店街', activity: '散步購物 (12:50-14:30)', mapUrl: 'https://maps.google.com/?q=日本橋筋商店街' },
                { day: 'Day 1 - 大阪市區精華', time: '14:40', location: '心齋橋 & 道頓堀', activity: '晚餐及逛街 (至22:00)', mapUrl: 'https://maps.google.com/?q=心齋橋' },
                
                // --- Day 2 | 大阪環球影城 (USJ) ---
                { day: 'Day 2 - 大阪環球影城', time: '早餐', location: '麥當勞/便利商店', activity: '快速解決早餐', mapUrl: '' },
                { day: 'Day 2 - 大阪環球影城', time: '07:30', location: '天下茶屋 → USJ', activity: '搭乘電車 (約45分/¥480)', mapUrl: 'https://maps.google.com/?q=USJ+環球影城' },
                { day: 'Day 2 - 大阪環球影城', time: '08:30', location: 'USJ', activity: '全天遊玩 (Express Pass必買, 至19:00)', mapUrl: '' },
                { day: 'Day 2 - 大阪環球影城', time: '19:30', location: 'USJ → 天下茶屋', activity: '搭乘電車 (約50分/¥480)', mapUrl: '' },
                
                // --- Day 3 | 奈良一日遊 ---
                { day: 'Day 3 - 奈良一日遊', time: '早餐', location: '立ち食い烏龍麵 / パンの小屋', activity: '站內或外帶早餐', mapUrl: '' },
                { day: 'Day 3 - 奈良一日遊', time: '08:00', location: '天下茶屋 → 近鐵奈良', activity: '搭乘電車 (約60分/¥640)', mapUrl: 'https://maps.google.com/?q=奈良公園' },
                { day: 'Day 3 - 奈良一日遊', time: '09:10', location: '奈良公園 → 東大寺 → 春日大社', activity: '逛景點與餵鹿 (至12:00)', mapUrl: 'https://maps.google.com/?q=東大寺' },
                { day: 'Day 3 - 奈良一日遊', time: '12:30', location: '奈良町', activity: '午餐 (和食或茶屋)', mapUrl: 'https://maps.google.com/?q=奈良町' },
                { day: 'Day 3 - 奈良一日遊', time: '13:30', location: '奈良町', activity: '散步、買伴手禮 (至15:30)', mapUrl: '' },
                { day: 'Day 3 - 奈良一日遊', time: '16:00', location: '近鐵奈良 → 天下茶屋', activity: '搭乘電車 (約60分/¥640)', mapUrl: '' },
                
                // --- Day 4 | 京都一日深度遊 ---
                { day: 'Day 4 - 京都一日深度遊', time: '早餐', location: '志津屋 / 小川珈琲', activity: '雞蛋三明治或咖啡早餐', mapUrl: '' },
                { day: 'Day 4 - 京都一日深度遊', time: '06:30', location: '天下茶屋 → 京都', activity: '搭乘電車 (約70分/¥1,000)', mapUrl: 'https://maps.google.com/?q=京都站' },
                { day: 'Day 4 - 京都一日深度遊', time: '08:00', location: '伏見稻荷大社', activity: '參觀 (08:00-09:00)', mapUrl: 'https://maps.google.com/?q=伏見稻荷大社' },
                { day: 'Day 4 - 京都一日深度遊', time: '09:30', location: '清水寺＋和服體驗', activity: '提前預約 (至12:00)', mapUrl: 'https://maps.google.com/?q=清水寺' },
                { day: 'Day 4 - 京都一日深度遊', time: '12:15', location: '空禪寺', activity: '午餐 (需預約)', mapUrl: '' },
                { day: 'Day 4 - 京都一日深度遊', time: '13:20', location: '二年坂、三年坂', activity: '散步 (至14:20)', mapUrl: 'https://maps.google.com/?q=二年坂' },
                { day: 'Day 4 - 京都一日深度遊', time: '14:40', location: '祇園（花見小路）', activity: '參觀 (14:40-15:30)', mapUrl: 'https://maps.google.com/?q=祇園' },
                { day: 'Day 4 - 京都一日深度遊', time: '15:40', location: '錦市場', activity: '逛街 (15:40-16:40)', mapUrl: 'https://maps.google.com/?q=錦市場' },
                { day: 'Day 4 - 京都一日深度遊', time: '17:10', location: '金閣寺', activity: '參觀 (17:10-18:10)', mapUrl: 'https://maps.google.com/?q=金閣寺' },
                { day: 'Day 4 - 京都一日深度遊', time: '19:00', location: '京都 → 天下茶屋', activity: '搭乘電車 (約90分/¥1,000)', mapUrl: '' },
                
                // --- Day 5 | 大阪收尾 + 臨空城Outlet ---
                { day: 'Day 5 - 大阪收尾 + Outlet', time: '早餐', location: '喫茶Y / ドトールコーヒー', activity: '新世界或新今宮站早餐', mapUrl: '' },
                { day: 'Day 5 - 大阪收尾 + Outlet', time: '08:30', location: '通天閣（新世界）', activity: '參觀 (08:30-10:30)', mapUrl: 'https://maps.google.com/?q=通天閣' },
                { day: 'Day 5 - 大阪收尾 + Outlet', time: '11:00', location: '大阪上本町（近鐵百貨）', activity: '購物 (11:00-12:30)', mapUrl: 'https://maps.google.com/?q=大阪上本町' },
                { day: 'Day 5 - 大阪收尾 + Outlet', time: '13:00', location: '天神橋筋商店街', activity: '散步購物 (13:00-14:30)', mapUrl: 'https://maps.google.com/?q=天神橋筋商店街' },
                { day: 'Day 5 - 大阪收尾 + Outlet', time: '15:00', location: '天下茶屋 → 臨空城Outlet', activity: '搭乘電車 (約40分/¥930)', mapUrl: 'https://maps.google.com/?q=臨空城Outlet' },
                { day: 'Day 5 - 大阪收尾 + Outlet', time: '15:40', location: '臨空城Outlet', activity: '購物 (至20:00)', mapUrl: '' },
                { day: 'Day 5 - 大阪收尾 + Outlet', time: '20:00', location: '臨空城 → 關西機場', activity: '搭乘電車 (約10分/¥270)', mapUrl: 'https://maps.google.com/?q=關西機場' },
            ]);
            // --------------------------------------------------------
            
            const filteredSchedule = computed(() => {
                return scheduleData.filter(item => item.day === selectedDay.value)
            })

            const deleteItem = (index) => {
                // 找出原始索引並刪除
                const itemToDelete = filteredSchedule.value[index];
                const originalIndex = scheduleData.findIndex(item => 
                    item.day === itemToDelete.day && 
                    item.time === itemToDelete.time && 
                    item.location === itemToDelete.location
                );
                if (originalIndex !== -1) {
                    scheduleData.splice(originalIndex, 1);
                }
            }

            return { selectedDay, days, filteredSchedule, deleteItem }
        }
    }

    // --- 2. 分帳系統組件 (ExpenseSplitter) ---
    const ExpenseSplitter = {
        template: `
            <div class="space-y-4">
                <h2 class="text-2xl font-semibold text-gray-800">💰 分帳系統</h2>
                
                <div class="bg-blue-50 p-4 rounded-xl shadow-inner border border-blue-200">
                    <h3 class="font-bold text-lg text-blue-700 mb-2">新增支出</h3>
                    <div class="grid grid-cols-2 gap-3">
                        <input type="text" v-model="newExpense.description" placeholder="項目 (e.g. 午餐)" class="p-2 border rounded-lg focus:ring-blue-500 focus:border-blue-500">
                        <input type="number" v-model.number="newExpense.amount" placeholder="金額 (日圓)" class="p-2 border rounded-lg focus:ring-blue-500 focus:border-blue-500">
                        
                        <select v-model="newExpense.paidBy" class="col-span-1 p-2 border rounded-lg focus:ring-blue-500 focus:border-blue-500 bg-white">
                            <option value="">選擇付款人</option>
                            <option v-for="p in members" :value="p">{{ p }}</option>
                        </select>
                        
                        <button @click="addExpense" class="bg-green-500 text-white py-2 rounded-lg hover:bg-green-600 transition-colors font-medium">
                            記錄支出
                        </button>
                    </div>
                </div>

                <h3 class="font-bold text-xl text-gray-800 mt-6">結算結果 (誰該給誰錢)</h3>
                <div class="space-y-3">
                    <div v-if="settlements.length === 0" class="text-center p-4 bg-yellow-50 rounded-xl text-yellow-700">
                        尚未有需結算的項目。
                    </div>
                    <div v-for="(settle, index) in settlements" :key="index"
                         class="flex items-center justify-between p-3 bg-white rounded-lg shadow-sm border border-gray-100">
                        <p class="text-gray-700 font-medium">
                            <span class="font-bold text-red-500">{{ settle.from }}</span> 應付給 
                            <span class="font-bold text-green-500">{{ settle.to }}</span>
                        </p>
                        <span class="text-lg font-bold text-gray-800">¥ {{ settle.amount.toFixed(0) }}</span>
                    </div>
                </div>

                <h3 class="font-bold text-xl text-gray-800 mt-6">支出明細</h3>
                <ul class="space-y-2">
                    <li v-for="(expense, index) in expenses" :key="index"
                        class="flex justify-between items-center bg-gray-50 p-3 rounded-lg border-l-4 border-blue-400">
                        <div>
                            <p class="font-medium text-gray-800">{{ expense.description }}</p>
                            <p class="text-sm text-gray-500">付款人: {{ expense.paidBy }}</p>
                        </div>
                        <div class="text-right">
                            <span class="text-lg font-bold text-gray-800">¥ {{ expense.amount }}</span>
                            <button @click="deleteExpense(index)" class="ml-2 text-red-400 hover:text-red-600">
                                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                            </button>
                        </div>
                    </li>
                </ul>
            </div>
        `,
        setup() {
            const members = reactive(['小明', '小華', '小美']) // 範例成員
            const expenses = reactive([])
            const newExpense = reactive({ description: '', amount: null, paidBy: '' })

            const addExpense = () => {
                if (newExpense.description && newExpense.amount > 0 && newExpense.paidBy) {
                    expenses.push({ ...newExpense })
                    newExpense.description = ''
                    newExpense.amount = null
                    newExpense.paidBy = ''
                } else {
                    alert('請填寫完整的支出資訊！')
                }
            }
            
            const deleteExpense = (index) => {
                expenses.splice(index, 1);
            }

            const settlements = computed(() => {
                if (expenses.length === 0) return []

                const totalExpense = expenses.reduce((sum, e) => sum + e.amount, 0)
                const perPersonShare = totalExpense / members.length
                
                // 1. 計算每人的淨餘額 (正數: 應收, 負數: 應付)
                const memberBalances = {}
                members.forEach(m => memberBalances[m] = -perPersonShare)
                
                expenses.forEach(e => {
                    if (memberBalances[e.paidBy] !== undefined) {
                        memberBalances[e.paidBy] += e.amount
                    }
                })

                // 2. 轉換為可操作的陣列並排序 (收錢的人在前, 付錢的人在後)
                let balancesArray = members.map(name => ({
                    name: name,
                    balance: memberBalances[name]
                })).filter(b => Math.abs(b.balance) > 0.1) // 忽略接近零的餘額

                balancesArray.sort((a, b) => b.balance - a.balance)

                // 3. 結算邏輯
                const result = []
                let giverIndex = balancesArray.length - 1 // 從應付錢最多的人開始
                let receiverIndex = 0 // 從應收錢最多的人開始

                while (giverIndex > receiverIndex) {
                    const giver = balancesArray[giverIndex]
                    const receiver = balancesArray[receiverIndex]

                    const paymentAmount = Math.min(receiver.balance, Math.abs(giver.balance))
                    
                    if (paymentAmount > 0.1) {
                        result.push({
                            from: giver.name,
                            to: receiver.name,
                            amount: paymentAmount
                        })
                    }

                    receiver.balance -= paymentAmount
                    giver.balance += paymentAmount
                    
                    if (receiver.balance < 0.1) {
                        receiverIndex++ // 接收者結算完畢
                    }

                    if (giver.balance > -0.1) {
                        giverIndex-- // 支付者結算完畢
                    }
                }

                return result
            })

            return { members, expenses, newExpense, addExpense, deleteExpense, settlements }
        }
    }

    // --- 3. 地圖組件 (Map) ---
    const Map = {
        template: `
            <div class="space-y-4">
                <h2 class="text-2xl font-semibold text-gray-800">🗺 地圖與導航 (模擬)</h2>
                <p class="text-sm text-gray-500">在實際開發中，這裡會載入 Google Maps 或 Leaflet 等地圖服務。</p>
                <div class="bg-gray-200 h-96 rounded-xl border border-gray-300 flex items-center justify-center">
                    <div class="text-center text-gray-600">
                        <svg class="w-10 h-10 mx-auto mb-2 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 20l-5.447-2.723A1 1 0 013 16.382V5.618a1 1 0 01.553-.894L9 2m9 18l5.447-2.723A1 1 0 0021 16.382V5.618a1 1 0 00-.553-.894L15 2m0 0l-1 1h-2l-1-1m0 0l-1 1H9l-1-1m4 0V2h2v4"></path></svg>
                        <p class="font-medium">地圖服務載入區</p>
                        <p class="text-xs mt-1">模擬：顯示您行程中的地點</p>
                    </div>
                </div>
                
                <h3 class="font-bold text-xl text-gray-800 mt-6">快速導航</h3>
                <div class="space-y-3">
                    <button class="w-full text-left p-3 bg-white border-l-4 border-red-500 rounded-lg shadow hover:bg-red-50/50 transition-colors">
                        <p class="font-medium">🔴 前往 大阪城 (點擊模擬導航)</p>
                        <p class="text-xs text-gray-500">Day 1 行程</p>
                    </button>
                    <button class="w-full text-left p-3 bg-white border-l-4 border-purple-500 rounded-lg shadow hover:bg-purple-50/50 transition-colors">
                        <p class="font-medium">🟣 前往 伏見稻荷大社 (點擊模擬導航)</p>
                        <p class="text-xs text-gray-500">Day 4 行程</p>
                    </button>
                </div>

            </div>
        `,
        // 在實際應用中，您會在這裡使用 Google Maps API 或其他地圖庫
        setup() {
            return {}
        }
    }

    // --- 4. 根應用 (App) ---
    createApp({
        components: {
            TripPlanner,
            ExpenseSplitter,
            Map
        },
        setup() {
            // 用來控制底部導航欄的狀態
            const currentView = ref('planner') 

            return {
                currentView
            }
        }
    }).mount('#app')
</script>

</body>
</html>
