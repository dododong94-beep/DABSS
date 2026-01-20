<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>현장 작업 및 자재 관리 시스템</title>
    <!-- Tailwind CSS 로드 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons 로드 (아이콘 사용) -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        body { font-family: 'Inter', sans-serif; }
        /* 입력 섹션의 그리드 레이아웃 */
        .input-grid {
            display: grid;
            grid-template-columns: 1fr;
            gap: 1.5rem;
        }
        @media (min-width: 768px) {
            .input-grid {
                grid-template-columns: repeat(2, 1fr);
            }
        }
        /* 자재 반입 타임라인 스타일 (테이블 레이아웃으로 변경) */
        .sticky-col {
            position: sticky;
            left: 0;
            z-index: 10;
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">

    <div class="container mx-auto p-4 md:p-8">
        <header class="mb-8">
            <h1 class="text-4xl font-extrabold text-indigo-800 text-center mb-2">HS 어울림 청주사직 DAB's 회의</h1>
            <p class="text-center text-gray-600">작업 내용 및 자재 반입 일정을 실시간으로 입력하고 확인하세요.</p>
            
            <!-- [추가된 문구] 다음 날짜 입력 안내 -->
            <p id="next-day-notice" class="text-center text-sm font-semibold text-red-600 mt-2 mb-4">
                <!-- JavaScript에서 내용이 채워집니다. -->
            </p>
            
            <div id="auth-status" class="text-sm text-center mt-2 text-gray-500">
                <span id="user-id-display">사용자 ID: 로딩 중...</span>
            </div>
        </header>

        <!-- 탭 메뉴 -->
        <nav class="flex space-x-2 bg-white p-2 rounded-lg shadow-md mb-8 max-w-lg mx-auto">
            <button id="tab-input" class="tab-button w-1/2 py-2 px-4 rounded-md text-sm font-medium transition-colors bg-indigo-600 text-white" onclick="showTab('input')">
                <i data-lucide="edit" class="inline-block w-4 h-4 mr-1 align-text-bottom"></i> 작업/자재 입력
            </button>
            <button id="tab-overview" class="tab-button w-1/2 py-2 px-4 rounded-md text-sm font-medium transition-colors text-gray-600 hover:bg-gray-100" onclick="showTab('overview')">
                <i data-lucide="layout-dashboard" class="inline-block w-4 h-4 mr-1 align-text-bottom"></i> 종합 일정 보기 (관리자)
            </button>
        </nav>

        <!-- 콘텐츠 영역 -->
        <main>
            <!-- 1. 입력 탭 -->
            <section id="content-input" class="tab-content input-grid">
                
                <!-- 동별 작업 내용 입력 폼 -->
                <div class="bg-white p-6 rounded-xl shadow-lg border border-indigo-100">
                    <h2 class="text-2xl font-semibold text-gray-700 mb-4 flex items-center">
                        <i data-lucide="hard-hat" class="w-6 h-6 mr-2 text-indigo-500"></i> 동별 작업 입력
                    </h2>
                    <form id="work-form" class="space-y-4">
                        
                        <!-- 공구 선택 -->
                        <div>
                            <label for="work-gonggu" class="block text-sm font-medium text-gray-700">공구 선택</label>
                            <select id="work-gonggu" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 p-2 border">
                                <option value="">--- 공구 선택 ---</option>
                            </select>
                        </div>
                        
                        <!-- 직종 선택 -->
                         <div>
                            <label for="work-job-type" class="block text-sm font-medium text-gray-700">직종 선택</label>
                            <select id="work-job-type" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 p-2 border">
                                <option value="">--- 직종 선택 ---</option>
                            </select>
                        </div>

                        <!-- 작업 동 선택 (공구에 따라 동적 변경) -->
                        <div>
                            <label for="work-dong" class="block text-sm font-medium text-gray-700">작업 동 선택</label>
                            <select id="work-dong" required disabled class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 p-2 border bg-gray-50">
                                <option value="">--- 먼저 공구를 선택하세요 ---</option>
                            </select>
                        </div>
                        
                        <!-- 업체명 입력란 -->
                        <div>
                            <label for="work-subcontractor" class="block text-sm font-medium text-gray-700">업체명</label>
                            <input type="text" id="work-subcontractor" required placeholder="예: 현대건설" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 p-2 border">
                        </div>
                        
                        <div>
                            <label for="work-content" class="block text-sm font-medium text-gray-700">작업 내용 (상세)</label>
                            <textarea id="work-content" rows="3" required placeholder="예: 콘크리트 타설 (동 입력 X)" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 p-2 border"></textarea>
                        </div>
                        <button type="submit" class="w-full bg-indigo-600 text-white font-bold py-2.5 rounded-lg hover:bg-indigo-700 transition-colors shadow-md">
                            작업 내용 저장
                        </button>
                    </form>
                </div>

                <!-- 시간대별 자재 반입 일정 입력 폼 -->
                <div class="bg-white p-6 rounded-xl shadow-lg border border-yellow-100">
                    <h2 class="text-2xl font-semibold text-gray-700 mb-4 flex items-center">
                        <i data-lucide="truck" class="w-6 h-6 mr-2 text-yellow-500"></i> 자재 반입 일정 입력
                    </h2>
                    <form id="material-form" class="space-y-4">
                        
                        <!-- 공구 선택 -->
                         <div>
                            <label for="material-gonggu" class="block text-sm font-medium text-gray-700">공구 선택</label>
                            <select id="material-gonggu" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                                <option value="">--- 공구 선택 ---</option>
                            </select>
                        </div>
                        
                        <!-- 반입 동 선택 (공구에 따라 동적 변경) -->
                        <div>
                            <label for="material-dong" class="block text-sm font-medium text-gray-700">반입 동 선택</label>
                            <select id="material-dong" required disabled class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border bg-gray-50">
                                <option value="">--- 먼저 공구를 선택하세요 ---</option>
                            </select>
                        </div>
                        
                        <!-- 게이트 선택 -->
                        <div>
                            <label for="material-gate" class="block text-sm font-medium text-gray-700">게이트 선택</label>
                            <select id="material-gate" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                                <option value="">--- 게이트 선택 ---</option>
                            </select>
                        </div>
                        
                        <!-- 업체명 입력란 -->
                         <div>
                            <label for="material-subcontractor" class="block text-sm font-medium text-gray-700">업체명</label>
                            <input type="text" id="material-subcontractor" required placeholder="예: 현대건설" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                        </div>

                        <div>
                            <label for="material-name" class="block text-sm font-medium text-gray-700">자재명</label>
                            <input type="text" id="material-name" required placeholder="예: 레미콘 5m³" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                        </div>
                        
                        <!-- 차량/대수 입력란 -->
                         <div>
                            <label for="material-vehicle" class="block text-sm font-medium text-gray-700">차량/대수</label>
                            <input type="text" id="material-vehicle" required placeholder="예: 25톤 덤프 1대" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                        </div>
                        
                        <!-- 하역 장소 입력란 -->
                         <div>
                            <label for="material-unloading-location" class="block text-sm font-medium text-gray-700">하역장소</label>
                            <input type="text" id="material-unloading-location" required placeholder="예: 105동 앞 광장" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                        </div>

                        <div class="grid grid-cols-2 gap-4">
                             <div>
                                <label for="delivery-date" class="block text-sm font-medium text-gray-700">반입 날짜</label>
                                <input type="date" id="delivery-date" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                             </div>
                             <div>
                                <label for="delivery-time" class="block text-sm font-medium text-gray-700">반입 시간대</label>
                                <select id="delivery-time" required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 p-2 border">
                                    <option value="">--- 시간 선택 ---</option>
                                </select>
                             </div>
                        </div>
                        <button type="submit" class="w-full bg-yellow-600 text-white font-bold py-2.5 rounded-lg hover:bg-yellow-700 transition-colors shadow-md">
                            일정 저장
                        </button>
                    </form>
                </div>
            </section>

            <!-- 2. 종합 일정 보기 탭 (관리자 뷰) -->
            <section id="content-overview" class="tab-content hidden">
                <h2 class="text-3xl font-bold text-gray-800 mb-6">종합 일정표</h2>
                
                <!-- 작업 내용 종합 (동별 그룹화) -->
                <div class="mb-8">
                    <h3 class="text-xl font-semibold text-indigo-700 mb-3 border-b pb-2 flex items-center">
                        <i data-lucide="check-circle" class="w-5 h-5 mr-2"></i> 동별 작업 현황
                    </h3>
                    
                    <!-- 공구 필터 -->
                    <div class="mb-4">
                        <label for="overview-gonggu-filter" class="block text-sm font-medium text-gray-700">공구 필터</label>
                        <select id="overview-gonggu-filter" class="mt-1 block w-full max-w-xs rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 p-2 border">
                            <option value="ALL">전체 공구 보기</option>
                            <!-- Options loaded by JS -->
                        </select>
                    </div>

                    <!-- 동별 작업 현황 목록 (가로 배치) -->
                    <div id="work-overview-list" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                        <!-- 동별로 그룹화된 작업 항목이 여기에 실시간으로 로드됩니다. -->
                    </div>
                </div>

                <!-- 자재 반입 일정 종합 (타임 테이블 형식) -->
                <div>
                    <h3 class="text-xl font-semibold text-yellow-700 mb-3 border-b pb-2 flex items-center">
                        <i data-lucide="calendar" class="w-5 h-5 mr-2"></i> 자재 반입 일정
                    </h3>
                    <div id="material-overview-list" class="space-y-8">
                        <!-- 날짜/시간대별 매트릭스 형태로 로드됩니다. -->
                    </div>
                </div>
            </section>
        </main>
    </div>

    <!-- Firebase/Firestore/App Logic -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, onSnapshot, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // --- 현장 관련 상수 정의 ---
        
        const ALL_DONG_OPTIONS = ['101동', '102동', '103동', '104동', '105동', '106동', '107동', '108동', '109동', '110동', '111동', '112동', '113동', '114동', '115동', '116동', '117동'];
        
        const GONGGU_MAP = {
            '1공구': ['101동', '102동', '103동', '104동', '114동', '115동', '116동', '117동'],
            '2공구': ['105동', '106동', '107동', '108동', '109동', '110동', '111동', '112동', '113동']
        };
        const GONGGU_OPTIONS = Object.keys(GONGGU_MAP);
        
        const JOB_TYPES = ['건축', '전기/설비'];
        
        const TIME_SLOTS = [
            '07:00 - 08:00', '08:00 - 09:00', '09:00 - 10:00', 
            '10:00 - 11:00', '11:00 - 12:00', '12:00 - 13:00', 
            '13:00 - 14:00', '14:00 - 15:00', '15:00 - 16:00', 
            '16:00 - 17:00'
        ];
        
        const GATE_OPTIONS = ['1게이트', '3게이트', '7게이트'];
        
        // --- Firebase 및 상태 변수 ---
        
        let app;
        let db;
        let auth;
        let userId = null;
        let isAuthReady = false;

        let workListEl, materialListEl;
        let workTasksSnapshotData = []; // 작업 내용 전체 스냅샷 데이터를 저장할 배열
        let currentGongguFilter = 'ALL'; // 현재 선택된 공구 필터


        // --- 1. 유틸리티 함수 및 초기 설정 ---

        // Lucide 아이콘 초기화
        lucide.createIcons();

        // 탭 전환 함수
        window.showTab = function(tabName) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById(`content-${tabName}`).classList.remove('hidden');

            document.querySelectorAll('.tab-button').forEach(btn => {
                btn.classList.remove('bg-indigo-600', 'text-white');
                btn.classList.add('text-gray-600', 'hover:bg-gray-100');
            });
            document.getElementById(`tab-${tabName}`).classList.add('bg-indigo-600', 'text-white');
            document.getElementById(`tab-${tabName}`).classList.remove('text-gray-600', 'hover:bg-gray-100');
            lucide.createIcons(); // 아이콘 재생성
        }

        /**
         * 동적 드롭다운 업데이트 함수
         * @param {string} gonggu - 선택된 공구 이름 ('1공구' 또는 '2공구')
         * @param {string} dongSelectId - 업데이트할 동 드롭다운의 ID
         */
        function updateDongOptions(gonggu, dongSelectId) {
            const selectEl = document.getElementById(dongSelectId);
            selectEl.innerHTML = ''; // 기존 옵션 제거
            
            if (!gonggu || !GONGGU_MAP[gonggu]) {
                 selectEl.add(new Option('--- 먼저 공구를 선택하세요 ---', ''));
                 selectEl.disabled = true;
                 selectEl.classList.add('bg-gray-50');
                 return;
            }

            selectEl.add(new Option('--- 동 선택 ---', ''));
            
            const dongList = GONGGU_MAP[gonggu].sort();
            dongList.forEach(dong => {
                selectEl.add(new Option(dong, dong));
            });
            
            selectEl.disabled = false;
            selectEl.classList.remove('bg-gray-50');
        }

        // 옵션 로드 및 이벤트 리스너 설정
        function loadOptions() {
            const workGongguSelect = document.getElementById('work-gonggu');
            const materialGongguSelect = document.getElementById('material-gonggu');
            const workJobTypeSelect = document.getElementById('work-job-type');
            const deliveryTimeSelect = document.getElementById('delivery-time');
            const materialGateSelect = document.getElementById('material-gate');
            const overviewGongguFilter = document.getElementById('overview-gonggu-filter');
            
            // 공구 옵션 로드 (입력 폼)
            GONGGU_OPTIONS.forEach(gonggu => {
                workGongguSelect.add(new Option(gonggu, gonggu));
                materialGongguSelect.add(new Option(gonggu, gonggu));
            });
            
            // 공구 옵션 로드 (필터)
            GONGGU_OPTIONS.forEach(gonggu => {
                overviewGongguFilter.add(new Option(gonggu, gonggu));
            });

            // 직종 옵션 로드
            JOB_TYPES.forEach(type => {
                workJobTypeSelect.add(new Option(type, type));
            });

            // 시간 슬롯 로드
            TIME_SLOTS.forEach(slot => {
                deliveryTimeSelect.add(new Option(slot, slot));
            });

            // 게이트 옵션 로드
            GATE_OPTIONS.forEach(gate => {
                materialGateSelect.add(new Option(gate, gate));
            });
            
            // 동적 옵션 필터링 리스너 설정
            workGongguSelect.addEventListener('change', (e) => updateDongOptions(e.target.value, 'work-dong'));
            materialGongguSelect.addEventListener('change', (e) => updateDongOptions(e.target.value, 'material-dong'));

            // 종합 일정 보기 필터 리스너 설정
            overviewGongguFilter.addEventListener('change', (e) => {
                currentGongguFilter = e.target.value;
                renderWorkOverview(); // 필터 변경 시 작업 내용 다시 렌더링
            });

            // 오늘 날짜 기본 설정
            document.getElementById('delivery-date').valueAsDate = new Date();
        }

        // --- 2. Firebase 초기화 및 인증 ---

        async function initializeFirebase() {
            // ... (Firebase Initialization and Auth Logic remains the same)
            try {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
                const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
                const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        document.getElementById('user-id-display').textContent = `사용자 ID: ${userId.substring(0, 8)}... (전체 ID는 콘솔 확인)`;
                        isAuthReady = true;
                        setupRealtimeListeners(appId);
                    } else {
                        try {
                            if (initialAuthToken) {
                                await signInWithCustomToken(auth, initialAuthToken);
                            } else {
                                await signInAnonymously(auth);
                            }
                        } catch (error) {
                            console.error("인증 실패:", error);
                        }
                    }
                });

            } catch (error) {
                console.error("Firebase 초기화 또는 설정 에러:", error);
                document.getElementById('auth-status').textContent = 'Firebase 초기화 실패. 콘솔 확인 요망.';
            }
        }

        // --- 3. 데이터 저장 함수 ---

        // 작업 내용 저장
        document.getElementById('work-form').addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!userId) return console.warn("인증 대기 중...");

            const gonggu = document.getElementById('work-gonggu').value;
            const dong = document.getElementById('work-dong').value;
            const jobType = document.getElementById('work-job-type').value;
            const subcontractor = document.getElementById('work-subcontractor').value;
            const content = document.getElementById('work-content').value;
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

            try {
                await addDoc(collection(db, `artifacts/${appId}/public/data/work_tasks`), {
                    gonggu: gonggu,
                    job_type: jobType,
                    dong_name: dong,
                    subcontractor_name: subcontractor,
                    task_content: content,
                    timestamp: serverTimestamp(),
                    creator_id: userId,
                    status: 'In Progress'
                });
                
                // 폼 초기화 후 이전 공구/동/직종 선택 유지 (요청 사항 반영)
                document.getElementById('work-form').reset();
                document.getElementById('work-subcontractor').value = subcontractor; // 업체명 유지
                document.getElementById('work-gonggu').value = gonggu;
                document.getElementById('work-job-type').value = jobType; // 직종 유지
                updateDongOptions(gonggu, 'work-dong'); // 동 옵션 재로드
                document.getElementById('work-dong').value = dong;
                
                alertSuccess("작업 내용이 성공적으로 저장되었습니다!");
            } catch (error) {
                console.error("작업 내용 저장 중 오류:", error);
                alertError("작업 내용 저장 실패.");
            }
        });

        // 자재 일정 저장
        document.getElementById('material-form').addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!userId) return console.warn("인증 대기 중...");

            const gonggu = document.getElementById('material-gonggu').value;
            const dong = document.getElementById('material-dong').value;
            const gate = document.getElementById('material-gate').value;
            const subcontractor = document.getElementById('material-subcontractor').value;
            const material = document.getElementById('material-name').value;
            const vehicleCount = document.getElementById('material-vehicle').value; 
            const unloadingLocation = document.getElementById('material-unloading-location').value; 
            const date = document.getElementById('delivery-date').value;
            const time = document.getElementById('delivery-time').value;
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

            try {
                await addDoc(collection(db, `artifacts/${appId}/public/data/material_schedules`), {
                    gonggu: gonggu,
                    dong_name: dong,
                    gate: gate,
                    subcontractor_name: subcontractor,
                    material_name: material,
                    vehicle_count: vehicleCount,
                    unloading_location: unloadingLocation,
                    delivery_date: date,
                    delivery_time: time,
                    timestamp: serverTimestamp(),
                    creator_id: userId,
                });
                
                // 폼 초기화 후 이전 선택 유지 (요청 사항 반영)
                document.getElementById('material-form').reset();
                document.getElementById('material-subcontractor').value = subcontractor; // 업체명 유지
                document.getElementById('material-gonggu').value = gonggu;
                document.getElementById('material-gate').value = gate; // 게이트 유지
                document.getElementById('delivery-time').value = time; // 시간대 유지
                document.getElementById('material-name').value = material; // 자재명 유지
                document.getElementById('material-vehicle').value = vehicleCount; // 차량/대수 유지
                document.getElementById('material-unloading-location').value = unloadingLocation; // 하역장소 유지

                updateDongOptions(gonggu, 'material-dong');
                document.getElementById('material-dong').value = dong;
                document.getElementById('delivery-date').value = date;
                
                alertSuccess("자재 일정이 성공적으로 저장되었습니다!");
            } catch (error) {
                console.error("자재 일정 저장 중 오류:", error);
                alertError("자재 일정 저장 실패.");
            }
        });


        // --- 4. 실시간 데이터 표시 (관리자 뷰) ---
        
        /**
         * 작업 내용 데이터를 필터링하고 렌더링합니다.
         */
        function renderWorkOverview() {
            workListEl.innerHTML = ''; // 목록 초기화
            
            const tasksByDong = {};
            ALL_DONG_OPTIONS.forEach(dong => tasksByDong[dong] = []); 
            
            // 1. 데이터 필터링 및 그룹화
            const filteredTasks = currentGongguFilter === 'ALL'
                ? workTasksSnapshotData
                : workTasksSnapshotData.filter(task => task.gonggu === currentGongguFilter);

            filteredTasks.forEach(data => {
                const dong = data.dong_name;
                if (tasksByDong[dong]) { 
                    tasksByDong[dong].push(data);
                }
            });

            // 2. HTML 렌더링
            if (filteredTasks.length === 0) {
                 workListEl.innerHTML = `<div class="col-span-full text-gray-500 text-center p-4 bg-gray-100 rounded-lg">등록된 작업 내용이 없거나, 현재 공구에 해당하는 작업이 없습니다.</div>`;
            } else {
                 // 렌더링할 동 목록 결정 (ALL_DONG_OPTIONS 순서 유지)
                const dongsToRender = currentGongguFilter === 'ALL'
                    ? ALL_DONG_OPTIONS
                    : GONGGU_MAP[currentGongguFilter].sort((a, b) => parseInt(a) - parseInt(b));

                dongsToRender.forEach(dong => {
                    const tasks = tasksByDong[dong];

                    // 동별 컨테이너 생성
                    let dongHtml = `
                        <div class="border rounded-xl shadow-md overflow-hidden bg-white">
                            <h4 class="bg-indigo-600 text-white p-3 text-lg font-bold truncate">${dong} (${tasks.length}건)</h4>
                            <ul class="divide-y divide-gray-200">
                    `;
                    
                    if (tasks.length > 0) {
                        tasks.forEach(data => {
                            const time = data.timestamp ? new Date(data.timestamp.seconds * 1000).toLocaleTimeString('ko-KR', { hour: '2-digit', minute: '2-digit' }) : 'N/A';
                            
                            dongHtml += `
                                <li class="p-4">
                                    <p class="text-xs font-medium text-gray-500 mb-1">
                                        <span class="text-indigo-500 font-bold">${data.gonggu || 'N/A'} | ${data.job_type || 'N/A'}</span> | 
                                        업체: ${data.subcontractor_name || 'N/A'} | ${time}
                                    </p>
                                    <p class="text-base font-semibold text-gray-800">${data.task_content}</p>
                                </li>
                            `;
                        });
                    } else {
                         dongHtml += `
                            <li class="p-4 text-gray-400 text-sm italic">등록된 작업 내용이 없습니다.</li>
                        `;
                    }

                    dongHtml += `</ul></div>`;
                    workListEl.insertAdjacentHTML('beforeend', dongHtml);
                });
            }
           
            lucide.createIcons();
        }


        function setupRealtimeListeners(appId) {
            
            // 1. 작업 내용 리스너 (Firestore 변경 감지 시 workTasksSnapshotData 업데이트)
            const workRef = collection(db, `artifacts/${appId}/public/data/work_tasks`);
            const workQuery = query(workRef, orderBy('timestamp', 'desc')); 
            
            onSnapshot(workQuery, (snapshot) => {
                
                // Firestore에서 가져온 데이터를 workTasksSnapshotData에 저장
                workTasksSnapshotData = snapshot.docs.map(doc => doc.data());
                
                // 필터 상태를 반영하여 렌더링 함수 호출
                renderWorkOverview(); 

            }, (error) => {
                console.error("작업 내용 실시간 로드 오류:", error);
                workListEl.innerHTML = `<div class="col-span-full text-red-500 text-center p-4 bg-red-100 rounded-lg">데이터 로드 중 오류 발생: ${error.message.split('\n')[0]}</div>`;
            });

            // 2. 자재 일정 리스너 (게이트-시간 매트릭스 형식으로 변경)
            const materialRef = collection(db, `artifacts/${appId}/public/data/material_schedules`);
            const materialQuery = query(materialRef, orderBy('delivery_date', 'asc')); 
            
            onSnapshot(materialQuery, (snapshot) => {
                
                materialListEl.innerHTML = '';
                
                if (snapshot.empty) {
                    materialListEl.innerHTML = `<p class="text-gray-500 text-center p-4 bg-gray-100 rounded-lg">등록된 자재 일정이 없습니다.</p>`;
                    return;
                }

                // 데이터 그룹화: { [date]: { [time_slot]: { [gate]: [materials] } } }
                const schedulesByDate = {};
                snapshot.forEach(doc => {
                    const data = doc.data();
                    const date = data.delivery_date;
                    const timeSlot = data.delivery_time;
                    const gate = data.gate;

                    if (!schedulesByDate[date]) {
                        schedulesByDate[date] = {};
                    }
                    if (!schedulesByDate[date][timeSlot]) {
                        schedulesByDate[date][timeSlot] = {};
                        // 모든 게이트에 대한 빈 배열 초기화
                        GATE_OPTIONS.forEach(g => schedulesByDate[date][timeSlot][g] = []);
                    }
                    
                    if (schedulesByDate[date][timeSlot][gate]) {
                        schedulesByDate[date][timeSlot][gate].push(data);
                    }
                });
                
                // 날짜별 HTML 렌더링
                const sortedDates = Object.keys(schedulesByDate).sort();

                sortedDates.forEach(date => {
                    const dailySchedules = schedulesByDate[date];
                    
                    let dateHtml = `
                        <div class="mb-6 border rounded-xl shadow-lg bg-white overflow-x-auto">
                            <h4 class="bg-yellow-600 text-white p-3 text-xl font-bold rounded-t-xl">${date} 자재 반입 일정</h4>
                            
                            <!-- 타임 테이블 구조 -->
                            <table class="min-w-full divide-y divide-gray-200">
                                <thead class="bg-gray-50">
                                    <tr>
                                        <!-- 시간 (Sticky Column) -->
                                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider sticky-col bg-gray-50 border-r">시간</th>
                                        <!-- 게이트 (Columns) -->
                                        ${GATE_OPTIONS.map(gate => 
                                            `<th class="px-3 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">${gate}</th>`
                                        ).join('')}
                                    </tr>
                                </thead>
                                <tbody class="divide-y divide-gray-200">
                    `;
                    
                    // 시간대 (Rows) 렌더링
                    TIME_SLOTS.forEach(slot => {
                        const slotSchedules = dailySchedules[slot] || {}; // { '1게이트': [...], '3게이트': [...] }
                        const rowIsScheduled = GATE_OPTIONS.some(gate => slotSchedules[gate] && slotSchedules[gate].length > 0);
                        
                        dateHtml += `
                            <tr class="${rowIsScheduled ? 'bg-white hover:bg-yellow-50' : 'bg-gray-50 text-gray-400'}">
                                <!-- 시간 셀 -->
                                <td class="px-3 py-2 whitespace-nowrap text-sm font-medium sticky-col ${rowIsScheduled ? 'bg-white' : 'bg-gray-50'} border-r">${slot}</td>
                                
                                <!-- 게이트 셀 (Content) -->
                                ${GATE_OPTIONS.map(gate => {
                                    const materials = slotSchedules[gate] || [];
                                    
                                    if (materials.length > 0) {
                                        return `
                                            <td class="px-3 py-2 align-top text-xs border-l">
                                                ${materials.map(data => `
                                                    <div class="bg-yellow-100 p-1.5 rounded-md mb-1 shadow-sm border border-yellow-200 text-gray-800">
                                                        <p class="font-medium text-xs truncate">
                                                            [${data.gonggu || 'N/A'} - ${data.dong_name}] ${data.material_name}
                                                        </p>
                                                        <p class="text-[10px] text-gray-600 mt-0.5">
                                                            ${data.vehicle_count || 'N/A'} | 업체: ${data.subcontractor_name || 'N/A'}
                                                        </p>
                                                    </div>
                                                `).join('')}
                                            </td>
                                        `;
                                    } else {
                                        return `<td class="px-3 py-2 align-top text-xs text-center italic text-gray-300 border-l"></td>`;
                                    }
                                }).join('')}
                            </tr>
                        `;
                    });

                    dateHtml += `
                                </tbody>
                            </table>
                        </div>
                    `; 
                    materialListEl.insertAdjacentHTML('beforeend', dateHtml);
                });

                lucide.createIcons();
            }, (error) => {
                console.error("자재 일정 실시간 로드 오류:", error);
                 materialListEl.innerHTML = `<p class="text-red-500 text-center p-4 bg-red-100 rounded-lg">데이터 로드 중 오류 발생: ${error.message.split('\n')[0]}</p>`;
            });
        }
        
        // --- 5. Custom Alert 함수 (alert() 대체) ---

        function alertSuccess(message) {
            console.log("SUCCESS:", message);
            const body = document.body;
            const alertEl = document.createElement('div');
            alertEl.className = "fixed bottom-4 right-4 bg-green-500 text-white p-4 rounded-lg shadow-xl z-50 transition-all duration-300 transform translate-x-0";
            alertEl.innerHTML = `<i data-lucide="check" class="w-5 h-5 inline mr-2"></i>${message}`;
            body.appendChild(alertEl);
            lucide.createIcons();
            
            setTimeout(() => {
                alertEl.classList.add('translate-x-full');
                alertEl.addEventListener('transitionend', () => alertEl.remove());
            }, 3000);
        }

        function alertError(message) {
            console.error("ERROR:", message);
            const body = document.body;
            const alertEl = document.createElement('div');
            alertEl.className = "fixed bottom-4 right-4 bg-red-500 text-white p-4 rounded-lg shadow-xl z-50 transition-all duration-300 transform translate-x-0";
            alertEl.innerHTML = `<i data-lucide="x" class="w-5 h-5 inline mr-2"></i>${message}`;
            body.appendChild(alertEl);
            lucide.createIcons();

            setTimeout(() => {
                alertEl.classList.add('translate-x-full');
                alertEl.addEventListener('transitionend', () => alertEl.remove());
            }, 3000);
        }
        
        /**
         * 오늘 날짜를 기준으로 다음 날짜를 계산하고 포맷팅하여 표시합니다.
         */
        function displayNextDayNotice() {
            const nextDayNoticeEl = document.getElementById('next-day-notice');
            if (!nextDayNoticeEl) return;
            
            const today = new Date();
            const nextDay = new Date(today);
            nextDay.setDate(today.getDate() + 1);
            
            const year = nextDay.getFullYear();
            const month = nextDay.getMonth() + 1; // getMonth()는 0부터 시작
            const day = nextDay.getDate();

            // 예시: 26년 1월 23일 작업내용을 입력 바랍니다.
            nextDayNoticeEl.textContent = `${year}년 ${month}월 ${day}일 작업내용을 입력 바랍니다.`;
        }


        // --- 6. 앱 시작 ---
        document.addEventListener('DOMContentLoaded', () => {
            // DOM 요소 참조를 여기에 저장 (오류 방지)
            workListEl = document.getElementById('work-overview-list');
            materialListEl = document.getElementById('material-overview-list');

            // 핵심 요소가 누락되었을 경우를 대비한 최종 안전 체크
            if (!workListEl || !materialListEl) {
                console.error("ERROR: 필수 DOM 요소가 발견되지 않아 실시간 리스너를 설정할 수 없습니다.");
                return; 
            }

            loadOptions();
            initializeFirebase();
            displayNextDayNotice(); // 날짜 안내 문구 표시
            showTab('input'); // 초기 탭 설정
        });
    </script>
</body>
</html>
