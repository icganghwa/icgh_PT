<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>공개 강연 일정 관리 - 인천 강화 회중</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css">
  <style>
    :root {
      /* 메인 테마 블루 컬러 */
      --theme-blue: #4A74B5;
      --theme-blue-hover: #3b5d91;
    }

    body { background-color: #f8fafc; font-family: -apple-system, BlinkMacSystemFont, "Malgun Gothic", sans-serif; }
    
    /* 헤더 컬러 적용 */
    .card-header-custom { background-color: var(--theme-blue); color: white; }
    
    /* 테마 버튼 */
    .btn-theme { background-color: var(--theme-blue); color: white; border: none; transition: background-color 0.2s; }
    .btn-theme:hover { background-color: var(--theme-blue-hover); color: white; }
    
    /* 다크 차콜 포인트 컬러 */
    .btn-accent { background-color: #334155; color: white; border: none; transition: background-color 0.2s; }
    .btn-accent:hover { background-color: #1e293b; color: white; }
    
    /* 초기화 버튼 스타일 */
    .btn-reset { border-color: #ef4444; color: #ef4444; background-color: transparent; transition: all 0.2s; }
    .btn-reset:hover { background-color: #ef4444; color: white; }

    /* 골자 번호 텍스트 스타일 */
    .outline-text { font-size: 0.85rem; font-weight: 300; color: #475569; }
    
    /* 골자 번호 없는 제목 스타일 */
    .title-no-outline { font-weight: bold; font-style: italic; }
    
    .special-event { background-color: #f1f5f9; color: #64748b; }
    .empty-slot { background-color: #ffffff; color: #94a3b8; }

    /* 스크롤 테이블 및 고정 헤더 설정 */
    .table-scroll-container { max-height: 60vh; overflow-y: auto; }
    .table-scroll-container thead th {
      position: sticky; top: 0; z-index: 2;
      background-color: #f8f9fa; box-shadow: 0 1px 2px rgba(0,0,0,0.1);
      vertical-align: middle;
    }
    
    /* 로그인 배경 및 폼 스타일 */
    #loginScreen {
      background-color: #f1f5f9;
      position: fixed; top: 0; left: 0; right: 0; bottom: 0;
      z-index: 9999;
      display: flex;
    }
    .login-input {
      background-color: #f8fafc;
      border: 1px solid #e2e8f0;
      border-radius: 8px;
    }
    .login-input:focus {
      background-color: #ffffff;
      box-shadow: 0 0 0 3px rgba(74, 116, 181, 0.2);
      border-color: var(--theme-blue);
    }

    /* 테이블 내 줄바꿈을 위한 설정 */
    .lh-sm { line-height: 1.25; }

    /* 숫자 입력칸 화살표(스피너) 숨김 처리 (직접 입력 유도) */
    input[type="number"]::-webkit-outer-spin-button,
    input[type="number"]::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
    input[type="number"] {
      -moz-appearance: textfield;
    }
  </style>
</head>
<body>

<!-- 로그인 화면 -->
<div id="loginScreen" class="justify-content-center align-items-center">
  <div class="card shadow-sm border-0" style="width: 100%; max-width: 360px; border-radius: 16px; overflow: hidden;">
    <div class="text-center text-white py-4" style="background-color: var(--theme-blue);">
      <h3 class="fw-bold mb-1">공개 강연 일정 관리</h3>
      <p class="mb-0 opacity-75" style="font-size: 0.95rem;">인천 강화 회중</p>
    </div>
    <div class="card-body p-4 p-md-5 bg-white">
      <div class="mb-3">
        <label class="form-label fw-semibold text-secondary" style="font-size: 0.85rem;">아이디 (또는 이름)</label>
        <input type="text" id="loginId" class="form-control form-control-lg login-input fs-6" placeholder="아이디/이름을 입력하세요">
      </div>
      
      <div class="mb-4">
        <label class="form-label fw-semibold text-secondary" style="font-size: 0.85rem;">비밀번호</label>
        <input type="password" id="loginPw" class="form-control form-control-lg login-input fs-6" placeholder="비밀번호를 입력하세요" onkeypress="handleEnter(event)">
      </div>
      
      <button class="btn btn-theme btn-lg w-100 fw-bold" style="border-radius: 8px;" onclick="doLogin()">로그인</button>
    </div>
  </div>
</div>

<!-- 메인 애플리케이션 화면 -->
<div id="mainApp" class="py-4" style="display: none;">
  <div class="container-fluid px-4">
    <!-- 최상단 타이틀 영역 (가운데 정렬) -->
    <div class="mb-4 pb-2 border-bottom text-center">
      <h2 class="fw-bold mb-1" style="color: var(--theme-blue);"><i class="bi bi-calendar-week me-2"></i>공개 강연 일정 관리</h2>
      <p class="text-muted mb-0">강연 제목과 초대 연사를 손쉽게 관리하세요.</p>
    </div>

    <!-- 1. 공개 강연 일정 목록 뷰 (기본 화면) -->
    <div id="talkListView" class="card shadow-sm border-0 mb-4">
      <div class="card-header card-header-custom py-3 d-flex justify-content-between align-items-center">
        <h5 class="mb-0 fw-semibold"><i class="bi bi-table me-2"></i>공개 강연 계획표</h5>
        <div class="text-end" style="line-height: 1.2;">
          <div class="fw-semibold text-light opacity-75" style="font-size: 1rem;">인천 강화 회중</div>
          <div id="roleBadgeStr" class="fw-bold mt-1" style="font-size: 0.85rem; color: #ffd700;"></div>
        </div>
      </div>
      
      <!-- 상단 툴바 (검색창 + 내보내기/사용자관리/로그아웃) -->
      <div class="card-body bg-white border-bottom py-3">
        <div class="row g-3 align-items-center">
          <div class="col-md-6 col-lg-7">
            <div class="input-group">
              <span class="input-group-text bg-light border-end-0"><i class="bi bi-search"></i></span>
              <input type="text" id="searchInput" class="form-control border-start-0 ps-0" placeholder="제목, 연사명, 회중, 사회, 낭독, 기도, 마이크 이름 검색..." onkeyup="renderTable()">
            </div>
          </div>
          <!-- 데스크탑 우측 정렬, 모바일 좌측 정렬 및 자동 줄바꿈(flex-wrap) 적용 -->
          <div class="col-md-6 col-lg-5 d-flex justify-content-md-end justify-content-start align-items-center gap-2 flex-wrap" id="topToolbar">
            <!-- JS 동적 렌더링 -->
          </div>
        </div>
      </div>

      <!-- 테이블 본문 -->
      <div class="table-responsive table-scroll-container">
        <table class="table table-hover align-middle mb-0 text-center" style="min-width: 1000px;">
          <thead class="table-light">
            <tr>
              <th style="width: 95px; white-space: nowrap;">날짜</th>
              <th style="width: 70px;">번호</th>
              <th class="text-center">제목</th>
              <th style="width: 90px;">연사</th>
              <th style="width: 100px;">회중</th>
              <th style="width: 70px;">사회</th>
              <th style="width: 70px;">낭독</th>
              <th style="width: 70px;">기도</th>
              <th style="width: 80px;">마이크</th>
              <th id="manageHeader" style="width: 90px; display: none;">관리</th>
            </tr>
          </thead>
          <tbody id="lectureTableBody">
            <!-- 동적 렌더링 -->
          </tbody>
        </table>
      </div>

      <!-- 하단 툴바 (공개강연목록 / 일정추가 / 초기화) -->
      <div class="card-footer bg-white py-3 border-top d-flex justify-content-end gap-2 flex-wrap" id="bottomToolbar">
        <!-- JS 동적 렌더링 -->
      </div>
    </div>

    <!-- 2. 사용자 관리 및 형제 명단 관리 뷰 (관리자 전용) -->
    <div id="userManageView" class="card shadow-sm border-0" style="display: none;">
      <div class="card-header bg-dark text-white py-3 d-flex justify-content-between align-items-center">
        <h5 class="mb-0 fw-semibold"><i class="bi bi-gear-fill me-2"></i>사용자 및 담당자 관리</h5>
        <button class="btn btn-sm btn-light fw-bold" onclick="switchView('talkListView')"><i class="bi bi-arrow-return-left me-1"></i>돌아가기</button>
      </div>
      <div class="card-body bg-white p-4">
        
        <div class="row g-5">
          <!-- 좌측: 일반 사용자 로그인 권한 관리 -->
          <div class="col-md-6 border-end">
            <h6 class="fw-bold text-primary mb-3"><i class="bi bi-person-lock me-2"></i>로그인 허용 사용자 관리</h6>
            <p class="text-muted" style="font-size: 0.85rem;">여기에 등록된 이름과 공통 비밀번호(191435)로 일반 사용자가 로그인할 수 있습니다.</p>
            
            <div class="input-group mb-2">
              <input type="text" id="newUserName" class="form-control" placeholder="추가할 사용자 이름 (예: 홍길동)" oninput="checkUserValidation()" onkeypress="if(event.key === 'Enter') addUser()">
              <button class="btn btn-primary px-3" id="addUserBtn" onclick="addUser()"><i class="bi bi-plus-lg me-1"></i>추가</button>
            </div>
            <!-- 이름 불일치 시 경고 문구 -->
            <div id="userValidationWarning" class="text-danger fw-bold mb-4" style="font-size: 0.8rem; display: none;">
              <i class="bi bi-exclamation-circle me-1"></i>이름을 다시 확인해 주세요
            </div>
            <div class="mb-3"></div>

            <div class="table-responsive" style="max-height: 360px; overflow-y: auto;">
              <table class="table table-bordered table-hover align-middle text-center mb-0">
                <thead class="table-light" style="position: sticky; top: 0; z-index: 1;">
                  <tr>
                    <th>사용자 이름 (로그인 ID)</th>
                    <th style="width: 80px;">삭제</th>
                  </tr>
                </thead>
                <tbody id="userTableBody">
                  <!-- 동적 렌더링 -->
                </tbody>
              </table>
            </div>
          </div>

          <!-- 우측: 담당 형제 드롭다운 명단 관리 -->
          <div class="col-md-6">
            <h6 class="fw-bold text-success mb-3"><i class="bi bi-person-lines-fill me-2"></i>담당 형제 명단 관리 (드롭다운 용)</h6>
            <p class="text-muted" style="font-size: 0.85rem;">일정 추가/수정 창에서 사회, 낭독, 기도, 마이크 담당자를 선택할 수 있는 자동완성 목록입니다.</p>
            
            <div class="input-group mb-4">
              <input type="text" id="newBrotherName" class="form-control" placeholder="추가할 형제 이름 (예: 김철수)" onkeypress="if(event.key === 'Enter') addBrother()">
              <button class="btn btn-success px-3" onclick="addBrother()"><i class="bi bi-plus-lg me-1"></i>추가</button>
            </div>

            <div class="table-responsive" style="max-height: 400px; overflow-y: auto;">
              <table class="table table-bordered table-hover align-middle text-center mb-0">
                <thead class="table-light" style="position: sticky; top: 0; z-index: 1;">
                  <tr>
                    <th>형제 이름</th>
                    <th style="width: 80px;">삭제</th>
                  </tr>
                </thead>
                <tbody id="brotherTableBody">
                  <!-- 동적 렌더링 -->
                </tbody>
              </table>
            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</div>

<!-- 전체 공개 강연 목록 확인/관리 모달 -->
<div class="modal fade" id="outlineListModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered modal-dialog-scrollable">
    <div class="modal-content">
      <div class="modal-header" style="background-color: #f8f9fa;">
        <h5 class="modal-title fw-bold" style="color: var(--theme-blue);"><i class="bi bi-list-ol me-2"></i>전체 공개 강연 목록</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body p-0">
        <ul class="list-group list-group-flush" id="outlineListContainer">
          <!-- JS 렌더링 -->
        </ul>
      </div>
      <div class="modal-footer border-0 d-flex justify-content-end gap-2" style="background-color: #f8f9fa;">
        <button type="button" class="btn btn-outline-primary px-3 fw-bold" id="btnAddOutline" onclick="addNewOutline()" style="display: none;"><i class="bi bi-plus-lg me-1"></i>강연 골자 추가</button>
        <button type="button" class="btn btn-secondary px-4" data-bs-dismiss="modal">닫기</button>
      </div>
    </div>
  </div>
</div>

<!-- 데이터 초기화 확인 모달 -->
<div class="modal fade" id="resetModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered modal-sm">
    <div class="modal-content">
      <div class="modal-body text-center pt-4 pb-3">
        <h6 class="fw-bold mb-0">정말로 모든 데이터를 초기화 하겠습니까?</h6>
        <p class="text-muted mt-2 mb-0" style="font-size: 0.85rem;">오늘 이후의 모든 데이터가 지워집니다.<br>(과거 데이터는 유지됩니다)</p>
      </div>
      <div class="modal-footer justify-content-center pb-3 pt-0 border-0 gap-2">
        <button type="button" class="btn btn-secondary px-4" data-bs-dismiss="modal">취소</button>
        <button type="button" class="btn btn-danger px-4" onclick="executeReset()">확인</button>
      </div>
    </div>
  </div>
</div>

<!-- 강연 등록/수정 모달 -->
<div class="modal fade" id="lectureModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title fw-bold" id="modalTitle">강연 등록</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">
        <form id="lectureForm">
          <input type="hidden" id="editIndex">
          
          <div class="row g-2 mb-3">
            <div class="col-md-6">
              <label class="form-label fw-semibold">날짜</label>
              <input type="text" id="inputDate" class="form-control" placeholder="예: 09월 06일" oninput="handleDateInput()" required>
              <div id="sundayWarning" class="form-text text-danger mt-1" style="display: none; font-weight: bold; font-size: 0.8rem;">
                <i class="bi bi-exclamation-circle me-1"></i>이 날짜는 일요일이 아닙니다
              </div>
            </div>
            <div class="col-md-6">
              <label class="form-label fw-semibold" style="color: var(--theme-blue);">강연 골자 번호</label>
              <input type="number" id="inputOutlineNo" class="form-control" style="border-color: var(--theme-blue);" placeholder="번호 직접 입력" oninput="handleOutlineInput()" autocomplete="off">
            </div>
          </div>
          
          <div class="mb-3">
            <div id="suspendedWarning" class="form-text text-danger mb-1" style="display: none; font-weight: bold; font-size: 0.85rem;">
              <i class="bi bi-x-circle-fill me-1"></i>이 공개 강연 골자는 사용하지 않을 것입니다 (26년5월 광고 참고)
            </div>
            <div id="recentWarning" class="form-text text-danger mb-1" style="display: none; font-weight: bold; font-size: 0.85rem;"></div>
            
            <label class="form-label fw-semibold">제목</label>
            <input type="text" id="inputTitle" class="form-control" placeholder="강연 제목 입력" required>
            <div id="titleHelp" class="form-text text-success" style="display: none; font-size: 0.8rem;"><i class="bi bi-check-circle me-1"></i>번호에 맞는 제목 자동입력됨</div>
          </div>
          
          <div class="row g-2 mb-3 border-bottom pb-3">
            <div class="col">
              <label class="form-label fw-semibold">연사</label>
              <input type="text" id="inputSpeaker" class="form-control" placeholder="연사 성명">
            </div>
            <div class="col">
              <label class="form-label fw-semibold">회중</label>
              <input type="text" id="inputCongregation" class="form-control" placeholder="소속 회중">
            </div>
          </div>

          <!-- 담당자 자동완성용 DataList -->
          <datalist id="brothersDatalist"></datalist>

          <div class="row g-2 mb-3">
            <div class="col-4">
              <label class="form-label fw-semibold text-secondary" style="font-size: 0.9rem;">사회</label>
              <input type="text" id="inputChairman" class="form-control form-control-sm" placeholder="직접입력/선택" list="brothersDatalist" autocomplete="off">
            </div>
            <div class="col-4">
              <label class="form-label fw-semibold text-secondary" style="font-size: 0.9rem;">낭독</label>
              <input type="text" id="inputReader" class="form-control form-control-sm" placeholder="직접입력/선택" list="brothersDatalist" autocomplete="off">
            </div>
            <div class="col-4">
              <label class="form-label fw-semibold text-secondary" style="font-size: 0.9rem;">기도</label>
              <input type="text" id="inputPrayer" class="form-control form-control-sm" placeholder="직접입력/선택" list="brothersDatalist" autocomplete="off">
            </div>
          </div>
          <div class="row g-2">
            <div class="col-6">
              <label class="form-label fw-semibold text-secondary" style="font-size: 0.9rem;">마이크 1</label>
              <input type="text" id="inputMic1" class="form-control form-control-sm" placeholder="직접입력/선택" list="brothersDatalist" autocomplete="off">
            </div>
            <div class="col-6">
              <label class="form-label fw-semibold text-secondary" style="font-size: 0.9rem;">마이크 2</label>
              <input type="text" id="inputMic2" class="form-control form-control-sm" placeholder="직접입력/선택" list="brothersDatalist" autocomplete="off">
            </div>
          </div>
        </form>
      </div>
      <div class="modal-footer border-0">
        <button type="button" class="btn btn-secondary px-4" data-bs-dismiss="modal">취소</button>
        <button type="button" class="btn btn-theme px-4" onclick="saveLecture()">저장</button>
      </div>
    </div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script>
// === 시스템 변수 선언 ===
let currentUserRole = null; 
let currentUserName = "";
let modal;
let resetModal;
let outlineListModal;
let lectures = [];
let displayList = [];
let allowedUsers = []; 
let suspendedOutlines = []; 
let brotherNames = []; 

const defaultBrotherNames = ["강용호", "김가우", "김대성", "김선정", "김윤건", "김현유", "민경태", "박남섭", "박수현", "박인규", "서금모", "서성남", "서영석", "은지상", "이석재", "이정로", "이찬호", "임동수", "장세관", "정용기", "조건희", "조오진", "주범서", "지역환", "차재승", "한일봉"];

const defaultOutlineTitles = {
  1: "당신은 하느님을 얼마나 잘 아는가?", 2: "당신은 마지막 날의 생존자가 될 것인가?", 3: "여호와의 연합된 조직과 함께 전진하라", 4: "우리 주위 세계에 나타나 있는 하느님에 대한 증거", 5: "행복한 가정생활을 위한 확실한 조언", 
  6: "노아 시대의 홍수와 당신", 7: "“부드러운 자비의 아버지”를 본받으라", 8: "자신을 위해서가 아니라 하느님의 뜻을 행하기 위하여 생활함", 9: "하느님의 말씀을 듣고 행하는 사람이 되십시오", 10: "언제나 정직하게 말하고 행동하십시오", 
  11: "그리스도를 본받아 ‘세상에 속하지 마십시오’", 12: "하느님은 권위에 대한 우리의 생각을 중요하게 여기신다", 13: "성과 결혼에 대한 하느님의 견해", 14: "깨끗한 백성은 여호와께 영예가 된다", 15: "“모든 사람에게 선한 일을 하십시오”", 
  16: "하느님과 계속 가까워지십시오", 17: "자신이 가진 모든 것으로 하느님께 영광을 돌리십시오", 18: "여호와를 당신의 산성으로 삼으십시오", 19: "당신의 미래—어떻게 알 수 있는가?", 20: "지금은 하느님께서 세상을 통치하실 때인가?", 
  21: "왕국 마련 안에서 자신의 위치를 소중히 여기십시오", 22: "당신은 여호와의 마련을 통해 유익을 얻고 있는가?", 23: "우리의 삶에는 분명 목적이 있다", 24: "당신은 “값진 진주”를 발견했는가?", 25: "세상의 영을 물리치십시오!", 
  26: "하느님은 당신을 소중히 여기시는가?", 27: "결혼 생활의 행복한 출발", 28: "결혼 생활에서 존중심과 사랑을 나타내십시오", 29: "자녀를 기르는 일에 따르는 책임과 축복", 30: "가족 간의 의사소통을 개선하는 방법", 
  31: "당신은 영적 필요를 느끼는가?", 32: "일상생활의 염려에 어떻게 대처할 수 있는가?", 33: "정의로운 세상—과연 올 것인가?", 34: "당신은 생존을 위한 표를 받을 것인가?", 35: "당신도 영원히 살 수 있다!", 
  36: "현 생명이 인생의 전부인가?", 37: "하느님의 길로 걷는 것이 정말 유익한가?", 38: "세상 끝을 어떻게 생존할 수 있는가?", 39: "예수 그리스도—세상을 이기시는 분", 40: "가까운 미래에 일어날 일", 
  41: "‘그대로 서서 여호와의 구원을 보십시오’", 42: "사랑이 증오를 이길 수 있는가?", 43: "하느님이 요구하시는 것—우리에게 언제나 유익하다", 44: "예수의 가르침은 당신에게 어떤 유익을 줄 수 있는가?", 45: "생명에 이르는 길을 따르십시오", 
  46: "끝까지 확신을 굳게 유지하십시오", 47: "”좋은 소식을 믿으십시오”", 48: "그리스도인들의 충성—시험받고 있다", 49: "이 땅이 과연 다시 깨끗해질 수 있는가?", 50: "지혜로운 결정—어떻게 내릴 수 있는가?", 
  51: "진리가 당신의 생활을 변화시키고 있는가?", 52: "당신의 하느님은 누구인가?", 53: "당신의 생각은 하느님의 생각과 일치한가?", 54: "하느님과 그분의 약속에 대한 믿음을 길러 나가십시오", 55: "하느님 앞에서 어떻게 좋은 이름을 얻을 수 있는가?", 
  56: "우리가 신뢰할 수 있는 지도자는 누구인가?", 57: "박해를 견디는 일", 58: "누가 그리스도의 참제자인가?", 60: "당신의 삶의 목적은 무엇인가?", 
  61: "당신은 누구의 약속을 신뢰하는가?", 62: "어디에서 진정한 희망을 발견할 수 있는가?", 63: "진리를 찾을 수 있습니까?", 64: "당신은 ‘쾌락을 사랑하는 사람’이 될 것인가, ‘하느님을 사랑하는 사람’이 될 것인가?", 65: "분노가 가득한 세상에서 평화를 이루는 방법", 
  66: "당신은 수확하는 일에 참여할 것인가?", 67: "여호와의 말씀과 그분의 창조물에 대해 묵상하십시오", 68: "“계속 ··· 서로 기꺼이 용서하십시오”", 69: "왜 자기희생적인 사랑을 나타내야 하는가?", 70: "왜 하느님을 신뢰해야 하는가?", 
  71: "“깨어 있으십시오”—왜 그리고 어떻게?", 72: "사랑—참그리스도인 회중을 알아볼 수 있는 표", 73: "“지혜의 마음”을 얻으십시오", 74: "여호와께서는 우리를 살펴보고 계신다", 75: "개인 생활에서 여호와의 통치권을 지지하십시오", 
  76: "오늘날의 문제들에 대처하는 데 성서 원칙이 도움이 되는가?", 77: "“후대에 힘쓰십시오\"", 78: "기쁜 마음으로 여호와를 섬기십시오", 79: "당신은 하느님의 친구가 될 것인가, 세상의 친구가 될 것인가?", 80: "과학과 성경—당신은 어느 쪽에 희망을 두는가?", 
  81: "누가 제자 삼는 일을 할 자격이 있는가?", 83: "그리스도인은 십계명을 지켜야 하는가?", 84: "당신은 이 세계의 운명을 피할 것인가?", 85: "폭력적인 세상에서 전해지고 있는 좋은 소식", 
  86: "하느님께서 들으시는 기도", 87: "당신과 하느님과의 관계는 어떠한가?", 88: "성경의 표준에 따라 생활해야 할 이유", 89: "진리에 목마른 사람은 오십시오!", 90: "참생명을 얻기 위해 힘써 노력하십시오!", 
  91: "메시아의 임재와 그의 통치", 92: "세상사에서 종교의 역할", 93: "자연재해—언제 사라질 것인가?", 94: "참종교는 인간 사회의 필요를 충족시켜 준다", 95: "영매술에 속지 마십시오!", 
  96: "종교의 미래는 어떠할 것인가?", 97: "구부러진 세대 가운데서 나무랄 데 없는 상태를 유지함", 98: "“이 세상의 장면은 변하고 있다”", 99: "성경을 신뢰할 수 있는 이유", 100: "영원히 지속될 튼튼한 우정을 기르는 방법", 
  101: "여호와—\"위대한 창조주”", 102: "“예언의 말씀”에 주의를 기울이십시오", 103: "어떻게 진정한 기쁨을 누릴 수 있는가?", 104: "부모 여러분—여러분은 내화 재료로 건축하고 있습니까?", 105: "모든 환난 중에 위로를 받음", 
  106: "땅을 파멸시키는 일로 인해 오게 될 하느님의 보응", 107: "당신은 훈련받은 양심을 통해 유익을 얻고 있는가?", 108: "당신도 확신을 가지고 미래를 맞이할 수 있다!", 109: "하느님의 왕국은 가까웠다", 110: "하느님을 첫째 자리에 둘 때 가정 생활에서 성공할 수 있다", 
  111: "인류를 완전히 치료하는 일—어떻게 가능한가?", 112: "이기적인 세상에서 사랑을 나타내는 방법", 113: "청소년들은 어떻게 행복하고 성공적인 삶을 살 수 있는가?", 114: "하느님의 경이로운 창조물들을 인식함", 115: "사탄의 교활한 행위로부터 자신을 보호하십시오", 
  116: "친구를 지혜롭게 선택하라!", 117: "선으로 악을 이기는 방법", 118: "여호와의 관점에서 청소년을 바라봄", 119: "그리스도인은 세상과 분리되어 있다—그것이 유익한 이유", 120: "지금 하느님의 통치권에 복종해야 하는 이유", 
  121: "세계적인 형제들로 이루어진 조직과 함께 대재난에서 생존하십시오", 124: "성서의 저자가 하느님임을 확신할 수 있는 근거", 125: "인류에게 대속물이 필요한 이유", 
  126: "누가 구원을 받을 수 있는가?", 127: "사람이 죽으면 어떻게 되는가?", 128: "지옥은 실제로 불타는 고초의 장소인가?", 129: "삼위일체는 성경의 가르침인가?", 130: "땅은 영원히 있을 것이다", 
  131: "마귀에 맞서 굳게 서 있으십시오!", 132: "부활—죽음에 대한 승리!", 133: "인간의 기원—무엇을 믿느냐가 중요한가?", 134: "그리스도인은 안식일을 지켜야 하는가?", 135: "생명과 피의 신성함", 
  136: "하느님은 숭배에서 형상을 사용하는 것을 승인하시는가?", 137: "성서의 기적들은 실제로 일어났는가?", 138: "타락한 세상에서 건전한 정신으로 살라", 139: "과학 세계에서의 하느님의 지혜", 140: "예수 그리스도는 실제로 누구인가?", 
  141: "인간 창조물이 신음하는 일—언제 끝날 것인가?", 142: "여호와께 도피해야 하는 이유", 143: "모든 위로의 하느님을 신뢰하라", 144: "그리스도의 지도를 받는 충성스러운 회중", 145: "누가 우리 하느님 여호와와 같은가?", 
  146: "여호와를 찬양하기 위하여 교육을 사용하라", 147: "여호와의 구원의 능력을 신뢰하라", 148: "당신은 생명에 대한 하느님의 견해를 가지고 있는가?", 149: "당신은 하느님과 함께 걷고 있는가?", 150: "이 세상은 멸망될 것인가?", 
  151: "여호와는 자신의 백성을 위한 “안전한 산성”이시다", 152: "실제로 있을 아마겟돈—왜? 언제?", 153: "“외경스러운 날”을 가깝게 여기십시오!", 154: "저울에 달린 인간 통치", 155: "바빌론의 심판 시간은 도래하였는가?", 
  156: "심판 날—두려워할 때인가, 아니면 희망을 가질 때인가?", 157: "참그리스도인들이 하느님의 가르침을 단장하는 방법", 158: "용기를 내어 여호와를 신뢰하여라", 159: "위험한 세상에서 안전을 찾는 일", 160: "당신의 그리스도인 신분을 지키라!", 161: "예수께서는 왜 고난을 겪고 죽으셨는가?", 162: "어둠의 세상으로부터의 구출", 163: "왜 참하느님을 두려워해야 하는가?", 164: "하느님은 지금도 통제력을 행사하시는가?", 165: "당신은 누구의 가치관을 소중히 여기는가?", 
  166: "진정한 믿음이란 무엇이며 어떻게 나타낼 수 있는가?", 167: "무분별한 세상에서 지혜롭게 행동하라", 168: "이 혼란스러운 세상에서도 안전을 느낄 수 있다!", 169: "왜 성서의 인도를 받아야 하는가?", 170: "누가 인류를 통치할 자격이 있는가?", 
  171: "당신도 지금부터 영원히 평화로운 삶을 누릴 수 있다!", 172: "우리는 하느님 앞에서 어떤 신분을 가지고 있는가?", 173: "하느님의 관점에서 참종교가 과연 있는가?", 174: "하느님의 신세계—누가 들어갈 수 있는가?", 175: "성서가 하느님의 말씀이라는 증거는 무엇인가?", 
  176: "진정한 평화와 안전—언제 있을 것인가?", 177: "고난의 때에 어디에서 도움을 얻을 수 있는가?", 178: "충절의 길로 걸으라", 179: "세상의 환상적인 것을 멀리하고, 왕국의 실제적인 것을 추구하라", 180: "부활 희망이 우리 자신에게 실제적이어야 하는 이유", 
  181: "끝은 우리가 생각하는 것보다 가까운가?", 182: "하느님의 왕국이 지금 우리를 위해 하고 있는 일", 183: "무가치한 것들을 보지 말고 물리치십시오!", 184: "죽으면 모든 것이 끝나는가?", 185: "진리가 우리의 생활에 영향을 미치는가?", 
  186: "하느님의 행복한 백성과 연합하십시오", 187: "사랑의 하느님께서 왜 악을 허용하시는가?", 188: "당신은 여호와께 확신을 두고 있는가?", 189: "하느님과 함께 걸으면 지금부터 영원히 축복을 받는다", 190: "하느님께서 약속하신 완전하고 행복한 가정", 
  191: "사랑과 믿음이 세상을 이기는 방법", 192: "당신은 영원한 생명에 이르는 길을 걷고 있는가?", 193: "세계적 고난의 때에 있을 구출", 194: "경건한 지혜는 우리에게 어떻게 유익을 주는가?"
};

const initialData = [
  { date: "09월 06일", outlineNo: 3, title: "여호와의 연합된 조직과 함께 전진하라", speaker: "강용호", congregation: "인천 심곡", chairman: "서영석", reader: "김윤건", prayer: "강용호", mic1: "서금모", mic2: "김가우" },
  { date: "09월 13일", outlineNo: 68, title: "“계속 ··· 서로 기꺼이 용서하십시오”", speaker: "은지상", congregation: "고양 동부", chairman: "서성남", reader: "이정로", prayer: "박수현", mic1: "박남섭", mic2: "주범서" },
  { date: "09월 20일", outlineNo: "", title: "순회 방문", speaker: "", congregation: "", chairman: "박수현", reader: "", prayer: "민경태", mic1: "", mic2: "" },
  { date: "09월 27일", outlineNo: "", title: "특별 강연", speaker: "박수현", congregation: "인천 강화", chairman: "이정로", reader: "서영석", prayer: "박수현", mic1: "정용기", mic2: "장세관" },
  { date: "10월 04일", outlineNo: "", title: "지부 대표자와 함께하는 순회 대회 (10/3 토)", speaker: "", congregation: "", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" },
  { date: "10월 11일", outlineNo: 31, title: "당신은 영적 필요를 느끼는가?", speaker: "지역환", congregation: "제천 동부", chairman: "서금모", reader: "박수현", prayer: "조오진", mic1: "김가우", mic2: "서영석" },
  { date: "10월 18일", outlineNo: 69, title: "왜 자기희생적인 사랑을 나타내야 하는가?", speaker: "이찬호", congregation: "김포 동부", chairman: "박남섭", reader: "김선정", prayer: "임동수", mic1: "김선정", mic2: "서금모" },
  { date: "10월 25일", outlineNo: 136, title: "하느님은 숭배에서 형상을 사용하는 것을 승인하시는가?", speaker: "조건희", congregation: "검단 남부", chairman: "박수현", reader: "임동수", prayer: "이정로", mic1: "박남섭", mic2: "김가우" },
  { date: "11월 01일", outlineNo: 110, title: "하느님을 첫째 자리에 둘 때 가정 생활에서 성공할 수 있다", speaker: "이석재", congregation: "김포 마산", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" },
  { date: "11월 08일", outlineNo: 67, title: "여호와의 말씀과 그분의 창조물에 대해 묵상하십시오", speaker: "차재승", congregation: "김포 동부", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" },
  { date: "11월 15일", outlineNo: "", title: "지부 방문 특별 모임 (11/14 토요일)", speaker: "", congregation: "", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" },
  { date: "11월 22일", outlineNo: 174, title: "하느님의 신세계—누가 들어갈 수 있는가?", speaker: "김현유", congregation: "소사 남부", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" },
  { date: "11월 29일", outlineNo: 108, title: "당신도 확신을 가지고 미래를 맞이할 수 있다!", speaker: "김대성", congregation: "고양 백마", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" },
  { date: "12월 13일", outlineNo: 172, title: "우리는 하느님 앞에서 어떤 신분을 가지고 있는가?", speaker: "한일봉", congregation: "인천 송림", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" },
  { date: "12월 20일", outlineNo: 88, title: "성경의 표준에 따라 생활해야 할 이유", speaker: "박인규", congregation: "김포 마산", chairman: "", reader: "", prayer: "", mic1: "", mic2: "" }
];

// === 초기 설정 및 세션 처리 ===
document.addEventListener("DOMContentLoaded", () => {
  let savedLectures = localStorage.getItem('public_talks_final_v6');
  if (savedLectures) {
      lectures = JSON.parse(savedLectures);
  } else {
      lectures = JSON.parse(JSON.stringify(initialData));
  }

  let savedOutlines = localStorage.getItem('outline_titles_dynamic_v2');
  if (savedOutlines) {
      outlineTitles = JSON.parse(savedOutlines);
  } else {
      outlineTitles = JSON.parse(JSON.stringify(defaultOutlineTitles));
  }
  
  let savedSuspended = localStorage.getItem('suspended_outlines_v1');
  if (savedSuspended) {
      suspendedOutlines = JSON.parse(savedSuspended);
  }

  let savedUsers = localStorage.getItem('allowed_users_v1');
  if (savedUsers) {
      allowedUsers = JSON.parse(savedUsers);
  }
  
  let savedBrothers = localStorage.getItem('brothers_list_v1');
  if (savedBrothers) {
      brotherNames = JSON.parse(savedBrothers);
  } else {
      brotherNames = [...defaultBrotherNames];
  }
  updateDatalist(); 
  
  modal = new bootstrap.Modal(document.getElementById('lectureModal'));
  resetModal = new bootstrap.Modal(document.getElementById('resetModal'));
  outlineListModal = new bootstrap.Modal(document.getElementById('outlineListModal'));
  
  const savedRole = sessionStorage.getItem('userRole');
  const savedName = sessionStorage.getItem('userName');
  if(savedRole && savedName) {
      currentUserRole = savedRole;
      currentUserName = savedName;
      applyRoleUI();
  }
});

// === 로그인 로직 ===
function handleEnter(e) {
  if (e.key === 'Enter' || e.keyCode === 13) {
    doLogin();
  }
}

function doLogin() {
  const id = document.getElementById('loginId').value.trim();
  const pw = document.getElementById('loginPw').value.trim();

  if (id === 'icganghwa' && pw === '191475') {
      currentUserRole = 'ADMIN';
      currentUserName = '관리자';
  } else if (pw === '191435') {
      if (allowedUsers.includes(id)) {
          currentUserRole = 'USER';
          currentUserName = id;
      } else {
          alert('등록되지 않은 사용자 이름입니다. 관리자에게 문의하세요.');
          return;
      }
  } else {
      alert('아이디(이름) 또는 비밀번호가 일치하지 않습니다.');
      return;
  }

  sessionStorage.setItem('userRole', currentUserRole);
  sessionStorage.setItem('userName', currentUserName);
  applyRoleUI();
}

function doLogout() {
  currentUserRole = null;
  currentUserName = "";
  sessionStorage.removeItem('userRole');
  sessionStorage.removeItem('userName');
  document.getElementById('loginId').value = '';
  document.getElementById('loginPw').value = '';
  document.getElementById('loginScreen').style.display = 'flex';
  document.getElementById('mainApp').style.display = 'none';
  
  switchView('talkListView');
}

function applyRoleUI() {
  document.getElementById('loginScreen').style.display = 'none';
  document.getElementById('mainApp').style.display = 'block';

  const topToolbar = document.getElementById('topToolbar');
  const bottomToolbar = document.getElementById('bottomToolbar');
  const roleBadgeStr = document.getElementById('roleBadgeStr');
  const manageHeader = document.getElementById('manageHeader');
  
  document.getElementById('btnAddOutline').style.display = (currentUserRole === 'ADMIN') ? 'block' : 'none';

  if (currentUserRole === 'ADMIN') {
      topToolbar.innerHTML = `
        <button class="btn btn-outline-success fw-semibold px-3" onclick="exportToPDF()"><i class="bi bi-printer me-1"></i>내보내기</button>
        <button class="btn btn-outline-secondary fw-semibold px-3" onclick="switchView('userManageView')"><i class="bi bi-gear-fill me-1"></i>설정 및 관리</button>
        <button class="btn btn-dark px-3" onclick="doLogout()"><i class="bi bi-box-arrow-right me-1"></i>로그아웃</button>
      `;

      bottomToolbar.innerHTML = `
        <button class="btn btn-outline-primary px-3 shadow-sm" onclick="openOutlineModal()"><i class="bi bi-list-ol me-1"></i>공개 강연 목록</button>
        <button class="btn btn-accent px-3 shadow-sm" onclick="openModal()"><i class="bi bi-plus-lg me-1"></i>일정 추가</button>
        <button class="btn btn-reset px-3 shadow-sm" onclick="openResetModal()"><i class="bi bi-arrow-counterclockwise me-1"></i>초기화</button>
      `;

      roleBadgeStr.innerHTML = `<i class="bi bi-shield-lock-fill me-1"></i>관리자 모드`;
      roleBadgeStr.style.color = "#ffd700";
      manageHeader.style.display = 'table-cell';
  } else {
      topToolbar.innerHTML = `
        <button class="btn btn-outline-success fw-semibold px-3" onclick="exportToPDF()"><i class="bi bi-printer me-1"></i>내보내기</button>
        <button class="btn btn-dark fw-semibold px-3" onclick="doLogout()"><i class="bi bi-box-arrow-right me-1"></i>로그아웃</button>
      `;

      bottomToolbar.innerHTML = `
        <button class="btn btn-outline-primary px-3 shadow-sm" onclick="openOutlineModal()"><i class="bi bi-list-ol me-1"></i>공개 강연 목록</button>
      `;

      roleBadgeStr.innerHTML = `<i class="bi bi-person-fill me-1"></i>${currentUserName} 형제`;
      roleBadgeStr.style.color = "#e2e8f0"; 
      manageHeader.style.display = 'none';
  }
  
  renderTable();
}

// === 자동완성(Datalist) 업데이트 함수 ===
function updateDatalist() {
  const datalist = document.getElementById('brothersDatalist');
  datalist.innerHTML = '';
  [...brotherNames].sort().forEach(name => {
    const option = document.createElement('option');
    option.value = name;
    datalist.appendChild(option);
  });
}

// === 전체 공개 강연 목록 관리 ===
function saveOutlineData() {
  localStorage.setItem('outline_titles_dynamic_v2', JSON.stringify(outlineTitles));
}

function saveSuspendedOutlines() {
  localStorage.setItem('suspended_outlines_v1', JSON.stringify(suspendedOutlines));
}

function openOutlineModal() {
  const container = document.getElementById('outlineListContainer');
  container.innerHTML = '';
  
  const keys = Object.keys(outlineTitles).map(Number).sort((a, b) => a - b);
  
  keys.forEach(num => {
    const isSuspended = suspendedOutlines.includes(num);
    const textClass = isSuspended ? 'text-danger fw-bold' : 'text-dark fw-medium';
    const badgeClass = isSuspended ? 'bg-danger' : 'bg-secondary';
    
    let actionBtns = '';
    if (currentUserRole === 'ADMIN') {
        const suspendBtn = `<button class="btn btn-sm ${isSuspended ? 'btn-danger' : 'btn-outline-danger bg-white'} border px-2 py-1 ms-1" onclick="toggleSuspend(${num})" title="사용 중지 토글">${isSuspended ? '<i class="bi bi-arrow-clockwise"></i> 재개' : '<i class="bi bi-slash-circle"></i> 중지'}</button>`;
        const editBtn = `<button class="btn btn-sm btn-light text-secondary border px-2 py-1 ms-2" onclick="editOutlineTitle(${num})" title="제목 수정"><i class="bi bi-pencil"></i></button>`;
        
        actionBtns = `
          <div class="d-flex flex-nowrap align-items-center">
            ${editBtn}
            ${suspendBtn}
          </div>
        `;
    }
    
    container.innerHTML += `
      <li class="list-group-item d-flex justify-content-between align-items-center py-2 px-3">
        <div class="d-flex align-items-center text-start" style="flex: 1;">
          <span class="badge ${badgeClass} me-3" style="min-width: 35px; font-size: 0.9rem;">${num}</span>
          <span class="${textClass} text-break">${outlineTitles[num]}</span>
        </div>
        ${actionBtns}
      </li>
    `;
  });
  outlineListModal.show();
}

function toggleSuspend(num) {
  const idx = suspendedOutlines.indexOf(num);
  if (idx > -1) {
      suspendedOutlines.splice(idx, 1); 
  } else {
      suspendedOutlines.push(num); 
  }
  saveSuspendedOutlines();
  openOutlineModal(); 
  renderTable(); 
}

function addNewOutline() {
  const numStr = prompt("새로 추가할 강연의 '골자 번호(숫자)'를 입력하세요:");
  if (!numStr) return;
  const num = parseInt(numStr, 10);
  
  if (isNaN(num)) {
      alert("유효한 숫자를 입력하세요.");
      return;
  }
  if (outlineTitles[num]) {
      alert("이미 존재하는 번호입니다. 기존 항목 우측의 '수정' 버튼을 이용해주세요.");
      return;
  }
  
  const title = prompt(`제 ${num}호 강연의 '제목'을 입력하세요:`);
  if (!title) return;
  
  outlineTitles[num] = title.trim();
  saveOutlineData();
  openOutlineModal(); 
}

function editOutlineTitle(num) {
  const currentTitle = outlineTitles[num];
  const newTitle = prompt(`제 ${num}호 강연의 새로운 제목을 입력하세요:`, currentTitle);
  
  if (newTitle !== null && newTitle.trim() !== '') {
      outlineTitles[num] = newTitle.trim();
      saveOutlineData();
      openOutlineModal(); 
  }
}

function switchView(viewId) {
  if (viewId === 'talkListView') {
    document.getElementById('talkListView').style.display = 'block';
    document.getElementById('userManageView').style.display = 'none';
    renderTable();
  } else if (viewId === 'userManageView') {
    document.getElementById('talkListView').style.display = 'none';
    document.getElementById('userManageView').style.display = 'block';
    renderUserManage();
    renderBrotherManage(); 
  }
}

// === 사용자 관리 로직 (숫자 번호 제거 및 이름/삭제만 표시) ===
function renderUserManage() {
  const tbody = document.getElementById('userTableBody');
  tbody.innerHTML = '';
  
  if (allowedUsers.length === 0) {
    tbody.innerHTML = `<tr><td colspan="2" class="text-muted py-4">등록된 사용자가 없습니다.</td></tr>`;
    return;
  }

  allowedUsers.forEach((name, idx) => {
    tbody.innerHTML += `
      <tr>
        <td class="fw-bold text-dark">${name}</td>
        <td><button class="btn btn-sm btn-outline-danger px-3" onclick="removeUser(${idx})"><i class="bi bi-trash"></i></button></td>
      </tr>
    `;
  });
}

// 사용자 추가 시 담당 형제 명단과 비교하여 중지/경고 처리
function checkUserValidation() {
  const nameInput = document.getElementById('newUserName').value.trim();
  const warningEl = document.getElementById('userValidationWarning');
  
  if (!nameInput) {
    warningEl.style.display = 'none';
    return;
  }
  
  // 담당 형제 명단(brotherNames)에 포함되어 있지 않다면 경고 표시
  if (!brotherNames.includes(nameInput)) {
    warningEl.style.display = 'block';
  } else {
    warningEl.style.display = 'none';
  }
}

function addUser() {
  const nameInput = document.getElementById('newUserName');
  const name = nameInput.value.trim();
  
  if (!name) return;
  
  // 만약 담당 형제 명단에 없다면 추가를 막고 경고
  if (!brotherNames.includes(name)) {
    alert("담당 형제 명단에 등록되지 않은 이름입니다. '이름을 다시 확인해 주세요'");
    return;
  }

  if (allowedUsers.includes(name)) {
    alert('이미 로그인 허용된 사용자입니다.');
    return;
  }
  
  allowedUsers.push(name);
  localStorage.setItem('allowed_users_v1', JSON.stringify(allowedUsers));
  nameInput.value = '';
  document.getElementById('userValidationWarning').style.display = 'none';
  renderUserManage();
}

function removeUser(idx) {
  if (confirm(`'${allowedUsers[idx]}' 사용자를 로그인 목록에서 삭제하시겠습니까?`)) {
    allowedUsers.splice(idx, 1);
    localStorage.setItem('allowed_users_v1', JSON.stringify(allowedUsers));
    renderUserManage();
  }
}

// === 담당 형제(드롭다운용) 명단 관리 로직 ===
function renderBrotherManage() {
  const tbody = document.getElementById('brotherTableBody');
  tbody.innerHTML = '';
  
  if (brotherNames.length === 0) {
    tbody.innerHTML = `<tr><td colspan="2" class="text-muted py-4">등록된 형제 명단이 없습니다.</td></tr>`;
    return;
  }

  const sortedBrothers = [...brotherNames].sort();
  sortedBrothers.forEach((name) => {
    const originalIdx = brotherNames.indexOf(name);
    tbody.innerHTML += `
      <tr>
        <td class="fw-bold text-dark">${name}</td>
        <td><button class="btn btn-sm btn-outline-danger px-3" onclick="removeBrother(${originalIdx})"><i class="bi bi-trash"></i></button></td>
      </tr>
    `;
  });
}

function addBrother() {
  const nameInput = document.getElementById('newBrotherName');
  const name = nameInput.value.trim();
  
  if (!name) return;
  if (brotherNames.includes(name)) {
    alert('이미 드롭다운 명단에 등록된 이름입니다.');
    return;
  }
  
  brotherNames.push(name);
  localStorage.setItem('brothers_list_v1', JSON.stringify(brotherNames));
  nameInput.value = '';
  renderBrotherManage();
  updateDatalist(); 
}

function removeBrother(idx) {
  const targetName = brotherNames[idx];
  if (confirm(`'${targetName}' 형제를 담당자 자동완성 목록에서 삭제하시겠습니까?`)) {
    brotherNames.splice(idx, 1);
    localStorage.setItem('brothers_list_v1', JSON.stringify(brotherNames));
    renderBrotherManage();
    updateDatalist(); 
  }
}

// === 시스템 기능 로직 ===
function saveToLocalStorage() {
  localStorage.setItem('public_talks_final_v6', JSON.stringify(lectures));
}

function parseToDateObj(dateStr) {
  const match = String(dateStr).match(/(\d{1,2})월\s*(\d{1,2})일/);
  if (!match) return null;
  const m = parseInt(match[1], 10) - 1;
  const d = parseInt(match[2], 10);
  const today = new Date();
  let y = today.getFullYear();
  if (today.getMonth() >= 8) { if (m < 8) y++; } 
  else { if (m >= 8) y--; }
  return new Date(y, m, d);
}

function getNextAugustRange() {
  const today = new Date();
  const currentMonth = today.getMonth();
  let firstDay = new Date(today.getFullYear(), currentMonth, 1);
  let firstSunday = new Date(firstDay);
  firstSunday.setDate(1 + (7 - firstDay.getDay()) % 7);
  let endDate = new Date(today.getFullYear() + 1, 7, 31, 23, 59, 59); 
  return { start: firstSunday, end: endDate };
}

function generateSundays(start, end) {
  let sundays = [];
  let current = new Date(start);
  while (current <= end) {
    let mStr = String(current.getMonth() + 1).padStart(2, '0');
    let dStr = String(current.getDate()).padStart(2, '0');
    sundays.push(`${mStr}월 ${dStr}일`);
    current.setDate(current.getDate() + 7);
  }
  return sundays;
}

function normalizeDateStr(dStr) {
  const m = String(dStr).match(/(\d{1,2})월\s*(\d{1,2})일/);
  if(m) return `${String(parseInt(m[1])).padStart(2,'0')}월 ${String(parseInt(m[2])).padStart(2,'0')}일`;
  return dStr;
}

function renderTable() {
  const tbody = document.getElementById('lectureTableBody');
  const search = document.getElementById('searchInput').value.trim().toLowerCase();

  displayList = [];
  const range = getNextAugustRange();
  const sundays = generateSundays(range.start, range.end);
  
  let windowLectures = lectures.map((l, idx) => ({ ...l, originalIndex: idx })).filter(l => {
    const dObj = parseToDateObj(l.date);
    if(!dObj) return true; 
    return dObj >= range.start && dObj <= range.end;
  });

  let coveredSet = new Set(windowLectures.map(l => normalizeDateStr(l.date)));
  sundays.forEach(sun => {
    if (!coveredSet.has(sun)) {
      windowLectures.push({ date: sun, outlineNo: "", title: "", speaker: "", congregation: "", chairman: "", reader: "", prayer: "", mic1: "", mic2: "", originalIndex: -1 });
    }
  });

  windowLectures.sort((a, b) => {
    const dA = parseToDateObj(a.date);
    const dB = parseToDateObj(b.date);
    if (!dA && !dB) return 0;
    if (!dA) return 1;
    if (!dB) return -1;
    return dA - dB;
  });
  displayList = windowLectures;

  tbody.innerHTML = '';

  displayList.forEach(item => {
    const searchableText = (
      (item.title || '') + 
      (item.speaker || '') + 
      (item.congregation || '') + 
      (item.outlineNo || '') + 
      (item.chairman || '') + 
      (item.reader || '') + 
      (item.prayer || '') + 
      (item.mic1 || '') + 
      (item.mic2 || '')
    ).toLowerCase();

    if (search === '' || searchableText.includes(search)) {
      const isPlaceholder = item.originalIndex === -1;
      const isSpecial = !isPlaceholder && !item.speaker && item.title;
      
      const tr = document.createElement('tr');
      if (isSpecial) tr.className = 'special-event';
      if (isPlaceholder) tr.className = 'empty-slot border-bottom';

      const isSuspendedOutline = item.outlineNo && suspendedOutlines.includes(Number(item.outlineNo));

      let titleClass = item.title ? (isSuspendedOutline ? 'text-danger fw-bold' : 'text-dark') : 'text-muted fst-italic';
      if (!item.outlineNo && item.title && !isPlaceholder) {
        titleClass += ' title-no-outline';
      }
      
      let titleHTML = item.title || (isPlaceholder ? '등록된 일정이 없습니다' : '(미정)');
      if (isSuspendedOutline) {
          titleHTML += `<div class="text-danger fw-bold mt-1" style="font-size: 0.8rem;"><i class="bi bi-exclamation-triangle-fill me-1"></i>이 공개 강연 골자는 사용하지 않을 것입니다 (26년5월 광고 참고)</div>`;
      }
      
      let outlineNoClass = isSuspendedOutline ? 'text-danger fw-bold' : 'outline-text';

      let micHTML = '';
      if (item.mic1 || item.mic2) {
          micHTML = [item.mic1, item.mic2].filter(Boolean).join('<br>');
      } else {
          micHTML = '<span class="text-muted">-</span>';
      }

      let manageCellHTML = '';
      if (currentUserRole === 'ADMIN') {
        if (isPlaceholder) {
          manageCellHTML = `
            <td>
              <button class="btn btn-sm btn-theme" style="width: 68px;" onclick="openModal(${item.originalIndex}, '${item.date}')" title="일정 추가">
                <i class="bi bi-plus-lg"></i> 추가
              </button>
            </td>
          `;
        } else {
          manageCellHTML = `
            <td>
              <div class="d-flex justify-content-center gap-1">
                <button class="btn btn-sm btn-outline-primary" style="width: 32px;" onclick="openModal(${item.originalIndex}, '${item.date}')" title="수정"><i class="bi bi-pencil"></i></button>
                <button class="btn btn-sm btn-outline-danger" style="width: 32px;" onclick="deleteLecture(${item.originalIndex})" title="삭제"><i class="bi bi-trash"></i></button>
              </div>
            </td>
          `;
        }
      }

      tr.innerHTML = `
        <td class="${isPlaceholder ? 'text-black-50' : 'text-secondary'} text-nowrap">${item.date || '-'}</td>
        <td>${item.outlineNo ? `<span class="${outlineNoClass}">${item.outlineNo}</span>` : `<span class="text-muted">-</span>`}</td>
        <td class="text-center ${titleClass}">${titleHTML}</td>
        <td>${item.speaker ? `<span>${item.speaker}</span>` : `<span class="text-muted">-</span>`}</td>
        <td>${item.congregation || '<span class="text-muted">-</span>'}</td>
        <td>${item.chairman || '<span class="text-muted">-</span>'}</td>
        <td>${item.reader || '<span class="text-muted">-</span>'}</td>
        <td>${item.prayer || '<span class="text-muted">-</span>'}</td>
        <td class="lh-sm">${micHTML}</td>
        ${manageCellHTML}
      `;
      tbody.appendChild(tr);
    }
  });
}

function openResetModal() { resetModal.show(); }

function executeReset() {
  const today = new Date();
  today.setHours(0, 0, 0, 0);
  lectures = lectures.filter(l => {
    const dObj = parseToDateObj(l.date);
    if (!dObj) return true; 
    return dObj < today;
  });
  saveToLocalStorage();
  resetModal.hide();
  renderTable(); 
}

function handleDateInput() { checkIfSunday(); checkRecentUsage(); }

function handleOutlineInput() { 
    autoFillTitle(); 
    checkRecentUsage(); 
    checkSuspendedUsage();
}

function checkSuspendedUsage() {
    const num = parseInt(document.getElementById('inputOutlineNo').value, 10);
    const warningEl = document.getElementById('suspendedWarning');
    if (num && suspendedOutlines.includes(num)) {
        warningEl.style.display = 'block';
    } else {
        warningEl.style.display = 'none';
    }
}

function checkIfSunday() {
  const dateStr = document.getElementById('inputDate').value.trim();
  const warningEl = document.getElementById('sundayWarning');
  if (!dateStr) { warningEl.style.display = 'none'; return; }
  const dateObj = parseToDateObj(dateStr);
  if (dateObj && dateObj.getDay() !== 0) warningEl.style.display = 'block';
  else warningEl.style.display = 'none';
}

function autoFillTitle() {
  const num = document.getElementById('inputOutlineNo').value;
  const titleInput = document.getElementById('inputTitle');
  const titleHelp = document.getElementById('titleHelp');
  if (num && outlineTitles[num]) {
    titleInput.value = outlineTitles[num];
    titleHelp.style.display = 'block';
  } else {
    titleHelp.style.display = 'none';
  }
}

function checkRecentUsage() {
  const num = document.getElementById('inputOutlineNo').value;
  const warningEl = document.getElementById('recentWarning');
  const currentIndex = document.getElementById('editIndex').value;
  const targetDateStr = document.getElementById('inputDate').value.trim();
  if (!num) { warningEl.style.display = 'none'; return; }

  let baseDate = targetDateStr ? parseToDateObj(targetDateStr) : new Date();
  if (!baseDate) baseDate = new Date();

  let mostRecentDateObj = null;
  let mostRecentDateStr = "";

  for (let i = 0; i < lectures.length; i++) {
    if (i.toString() === currentIndex) continue;
    if (String(lectures[i].outlineNo) === String(num) && lectures[i].date) {
      const otherDate = parseToDateObj(lectures[i].date);
      if (otherDate) {
        const diffDays = Math.abs(baseDate - otherDate) / (1000 * 60 * 60 * 24);
        if (diffDays <= 365) {
          if (!mostRecentDateObj || otherDate > mostRecentDateObj) {
            mostRecentDateObj = otherDate;
            mostRecentDateStr = lectures[i].date;
          }
        }
      }
    }
  }

  if (mostRecentDateObj) {
    warningEl.innerHTML = `<i class="bi bi-exclamation-triangle-fill me-1"></i>최근 1년 이내 (${mostRecentDateStr})에 사용된 연설 입니다.`;
    warningEl.style.display = 'block';
  } else {
    warningEl.style.display = 'none';
  }
}

function openModal(index = -1, dateStr = '') {
  document.getElementById('lectureForm').reset();
  document.getElementById('editIndex').value = index;
  document.getElementById('titleHelp').style.display = 'none';
  document.getElementById('recentWarning').style.display = 'none'; 
  document.getElementById('sundayWarning').style.display = 'none'; 
  document.getElementById('suspendedWarning').style.display = 'none';

  if (index !== -1 && index !== '-1') {
    document.getElementById('modalTitle').innerText = '강연 정보 수정';
    const item = lectures[index];
    document.getElementById('inputDate').value = item.date;
    document.getElementById('inputOutlineNo').value = item.outlineNo || '';
    document.getElementById('inputTitle').value = item.title;
    document.getElementById('inputSpeaker').value = item.speaker;
    document.getElementById('inputCongregation').value = item.congregation;
    
    document.getElementById('inputChairman').value = item.chairman || '';
    document.getElementById('inputReader').value = item.reader || '';
    document.getElementById('inputPrayer').value = item.prayer || '';
    document.getElementById('inputMic1').value = item.mic1 || '';
    document.getElementById('inputMic2').value = item.mic2 || '';
    
    checkIfSunday();
    checkSuspendedUsage();
  } else {
    document.getElementById('modalTitle').innerText = '강연 일정 등록';
    document.getElementById('inputDate').value = dateStr;
  }
  modal.show();
}

function saveLecture() {
  const index = document.getElementById('editIndex').value;
  const newTalk = {
    date: document.getElementById('inputDate').value.trim(),
    outlineNo: document.getElementById('inputOutlineNo').value.trim(),
    title: document.getElementById('inputTitle').value.trim(),
    speaker: document.getElementById('inputSpeaker').value.trim(),
    congregation: document.getElementById('inputCongregation').value.trim(),
    chairman: document.getElementById('inputChairman').value.trim(),
    reader: document.getElementById('inputReader').value.trim(),
    prayer: document.getElementById('inputPrayer').value.trim(),
    mic1: document.getElementById('inputMic1').value.trim(),
    mic2: document.getElementById('inputMic2').value.trim()
  };

  if(newTalk.outlineNo !== "") newTalk.outlineNo = Number(newTalk.outlineNo);

  if (!newTalk.date && !newTalk.title) {
    alert('날짜 또는 강연 제목을 입력해주세요.');
    return;
  }

  if (index === '-1' || index === '') lectures.push(newTalk);
  else lectures[index] = newTalk;

  saveToLocalStorage();
  modal.hide();
  renderTable();
}

function deleteLecture(index) {
  const dateStr = lectures[index].date || "해당";
  if (confirm(`'${dateStr}' 일정을 삭제하시겠습니까?`)) {
    lectures.splice(index, 1);
    saveToLocalStorage();
    renderTable();
  }
}

// === 새 창을 띄워 A4 문서 형식으로 PDF 저장/인쇄 기능 ===
function exportToPDF() {
  const printWindow = window.open('', '_blank');
  
  let html = `
  <html>
  <head>
    <title>공개 강연 계획표</title>
    <style>
      @page { size: A4 portrait; margin: 25mm 12mm 15mm 12mm; } 
      body { font-family: 'Malgun Gothic', sans-serif; padding: 0; margin: 0; color: #000; }
      h2 { text-align: center; margin-bottom: 5px; font-size: 16pt; font-weight: bold; color: #333; }
      .subtitle { text-align: right; margin-bottom: 4px; font-size: 9pt; font-weight: bold; color: #555; }
      table { width: 100%; border-collapse: collapse; font-size: 8.5pt; table-layout: fixed; line-height: 1.15; margin-top: 0px; margin-bottom: 20px; } 
      th, td { border: 1px solid #444; padding: 2px 1px; text-align: center; vertical-align: middle; word-break: keep-all; } 
      th { background-color: #f1f5f9; font-weight: bold; font-size: 8.5pt; padding: 4px 1px; height: 36px !important; }
      
      tr { height: 36px !important; } 
      
      .date-col { font-size: 7.5pt; white-space: nowrap; }
      .title-col { text-align: center; font-weight: normal; font-size: 10pt; letter-spacing: -0.5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
      
      .special-bold { font-weight: bold !important; color: #000 !important; }
      .special-event td { background-color: #f8fafc; }

      .page-break { page-break-after: always; }
    </style>
  </head>
  <body>
  `;

  // 데이터를 22개 단위로 분할(Chunking)
  let chunks = [];
  for (let i = 0; i < displayList.length; i += 22) {
      chunks.push(displayList.slice(i, i + 22));
  }

  chunks.forEach((chunk, chunkIndex) => {
    html += `
      <h2>공개 강연 계획표</h2>
      <div class="subtitle">인천 강화 회중</div>
      <table>
        <thead>
          <tr>
            <th style="width: 8%;">날짜</th>
            <th style="width: 44%;">제목</th>
            <th style="width: 8%;">연사</th>
            <th style="width: 10%;">회중</th>
            <th style="width: 7.5%;">사회</th>
            <th style="width: 7.5%;">낭독</th>
            <th style="width: 7.5%;">기도</th>
            <th style="width: 7.5%;">마이크</th>
          </tr>
        </thead>
        <tbody>
    `;

    chunk.forEach(item => {
      const isPlaceholder = item.originalIndex === -1;
      const isSpecial = !isPlaceholder && !item.outlineNo && item.title;
      
      let trClass = isSpecial ? ' class="special-event"' : '';
      let titleHTML = item.title || (isPlaceholder ? '' : '');
      let micHTML = [item.mic1, item.mic2].filter(Boolean).join('<br>') || '';
      let boldClass = isSpecial ? 'special-bold' : '';

      html += `
        <tr${trClass}>
          <td class="date-col ${boldClass}">${item.date || ''}</td>
          <td class="title-col ${boldClass}">${titleHTML}</td>
          <td>${item.speaker || ''}</td>
          <td>${item.congregation || ''}</td>
          <td>${item.chairman || ''}</td>
          <td>${item.reader || ''}</td>
          <td>${item.prayer || ''}</td>
          <td>${micHTML}</td>
        </tr>
      `;
    });

    html += `
        </tbody>
      </table>
    `;

    if (chunkIndex < chunks.length - 1) {
        html += `<div class="page-break"></div>`;
    }
  });

  html += `
  </body>
  </html>
  `;

  printWindow.document.write(html);
  printWindow.document.close();
  printWindow.focus();
  
  setTimeout(() => {
    printWindow.print();
  }, 250);
}
</script>
</body>
</html>
