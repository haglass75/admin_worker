<template>
  <div
    class="space-y-6 bg-white text-black dark:bg-black dark:text-white p-4 rounded">
    <!-- Font Awesome CDN 추가 -->
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" />

    <h1 class="text-3xl font-bold text-gray-800 dark:text-white">
      관리자 대시보드
    </h1>

    <!-- 통계 카드 -->

    <DashboardStats :stats="stats" />
    <!-- 예약 현황 -->
    <div
      class="bg-white dark:bg-gray-800 rounded-lg shadow text-gray-700 dark:text-gray-300">
      <!-- 검색 필터 -->
      <div class="p-4 border-b border-gray-200 dark:border-gray-700">
        <h2 class="text-lg font-semibold text-gray-800 dark:text-white mb-2">
          예약 현황
        </h2>
        <!-- space-y-4 -->
        <!-- 세로(y축)로 요소 사이에 1rem(=16px) 만큼 간격을 줍니다.
🧱 4. xl:space-y-0
xl: → 1200px 이상일 때만 적용 (Tailwind 기본은 1280px, 사용자 설정에 따라 1200px로 가능)
즉, 큰 화면에서는 space-y-0이 되어 세로 간격 제거
이유: 이제 가로 정렬로 바뀌니까 세로 간격은 필요 없음.
🧱 5. xl:flex-row
1200px 이상일 때는 가로(row) 방향으로 정렬
즉, 아래처럼 세로 → 가로로 바뀜
🧱 6. xl:items-center
가로 정렬 시 세로 가운데 정렬을 해줍니다.
즉, 높이가 다른 박스라도 세로 방향으로 중앙에 맞춰집니다.
🧱 7. xl:justify-between
가로로 나란히 배치된 요소들을 좌우로 일정하게 벌려서 정렬합니다.
→ 첫 번째 요소는 왼쪽 끝, 마지막 요소는 오른쪽 끝에 위치.
🧱 8. xl:space-x-4
가로(x축)로 요소 사이에 1rem(=16px) 만큼 간격을 줍니다.
세로 정렬 때는 space-y,
가로 정렬 때는 space-x를 쓰는 게 핵심입니다. -->
        <div
          class="flex flex-col space-y-4 xl:space-y-0 xl:flex-row xl:items-center xl:justify-between xl:space-x-4">
          <!-- 날짜 선택 -->
          <div
            class="flex flex-col space-y-2 xl:space-y-0 xl:flex-row xl:items-center xl:space-x-2">
            <label
              class="text-sm font-medium text-gray-700 dark:text-gray-300 whitespace-nowrap"
              >기준일</label
            >
            <div
              class="flex flex-col sm:flex-row items-center space-y-2 sm:space-y-0 sm:space-x-2">
              <input
                type="date"
                v-model="dateRange.start"
                class="w-full sm:w-auto px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white" />
              <span class="text-gray-500 dark:text-gray-400">~</span>
              <input
                v-model="dateRange.end"
                type="date"
                class="w-full sm:w-auto px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white" />
            </div>
          </div>

          <!-- 접수 구분 -->
          <!-- <div
            class="flex flex-col space-y-2 xl:space-y-0 xl:flex-row xl:items-center xl:space-x-2">
            <label
              class="text-sm font-medium text-gray-700 dark:text-gray-300 whitespace-nowrap"
              >접수구분</label
            >
            <select
              v-model="serviceType"
              class="w-full sm:w-auto px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white">
              <option value="all">전체</option>
              <option value="일반청소">일반청소</option>
              <option value="입주청소">입주청소</option>
              <option value="이사청소">이사청소</option>
            </select>
          </div> -->

          <!-- 접수 상태 -->
          <!-- <div
            class="flex flex-col space-y-2 xl:space-y-0 xl:flex-row xl:items-center xl:space-x-2">
            <label
              class="text-sm font-medium text-gray-700 dark:text-gray-300 whitespace-nowrap"
              >접수상태</label
            >
            <select
              v-model="receiptStatus"
              class="w-full sm:w-auto px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white">
              <option value="all">전체</option>
              <option value="예약완료">예약완료</option>
              <option value="진행중">진행중</option>
              <option value="대기중">대기중</option>
            </select>
          </div> -->
        </div>
      </div>
      <!-- 사용자 지정 props -->
      <!-- 예약 목록 (공통 컴포넌트 사용) 
      :data - 예약 목록 데이터
      :columns - 예약 목록 컬럼
      search-placeholder - 검색 플레이스홀더
      search-fields - 검색 필드
      filter-options - 필터 옵션
      filter-fn - 필터 함수
      items-per-page - 페이지당 아이템 수
      table-title - 테이블 제목
      total-label - 총 개수 라벨
      @row-click - 행 클릭 이벤트 
      -->
      <SearchTable
        :data="filteredReservaions"
        :columns="reservationColumns"
        search-placeholder="고객명 또는 예약번호로 검색..."
        :search-fields="['customerName', 'id']"
        :filter-options="dashboardFilterOptions"
        :filter-fn="dashboardFilterFn"
        :items-per-page="itemsPerPage"
        table-title="예약 목록"
        total-label="개"
        @row-click="handleRowClick" />
    </div>

    <Worker_dash />
    <!-- 차트와 최근 예약 -->
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <!-- 차트 -->
      <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-6">
        <h2 class="text-lg font-semibold text-gray-800 dark:text-white mb-4">
          예약 추이
        </h2>
        <div class="h-64">
          <Chart />
        </div>
      </div>

      <!-- 최근 예약 -->
      <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-6">
        <h2 class="text-lg font-semibold text-gray-800 dark:text-white mb-4">
          최근 예약
        </h2>
        <div class="space-y-4">
          <div
            v-for="reservation in recentReservations"
            :key="reservation.id"
            class="flex items-center justify-between p-4 bg-gray-50 dark:bg-gray-700 rounded-lg">
            <div>
              <p class="font-medium text-gray-900 dark:text-white">
                {{ reservation.customerName }}
              </p>
              <p class="text-sm text-gray-500 dark:text-gray-400">
                {{ reservation.date }}
              </p>
            </div>
            <span
              :class="getStatusClass(reservation.status)"
              class="px-2 py-1 text-xs font-semibold rounded-full">
              {{ reservation.status }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 예약 상세 모달 -->
  <div
    v-if="selectedReservation"
    class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center p-4 z-50">
    <div
      class="bg-white dark:bg-gray-800 rounded-lg shadow-xl max-w-4xl w-full max-h-[90vh] overflow-y-auto"
      @click.stop>
      <div class="p-6 border-b border-gray-200 dark:border-gray-700">
        <div class="flex justify-between items-center">
          <h3 class="text-lg font-medium text-gray-900 dark:text-white">
            예약 상세 정보
          </h3>
          <button
            @click="closeModal"
            class="text-gray-400 hover:text-gray-500 dark:hover:text-gray-300">
            <i class="fas fa-times"></i>
          </button>
        </div>
      </div>
      <div class="p-6">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <!-- 기본 정보 -->
          <div class="space-y-6">
            <div>
              <h4
                class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-2">
                기본 정보
              </h4>
              <div class="space-y-2">
                <div class="flex items-center">
                  <label
                    class="w-32 text-sm font-medium text-gray-700 dark:text-gray-300"
                    >예약번호</label
                  >
                  <span class="text-sm text-gray-900 dark:text-white">{{
                    selectedReservation.id
                  }}</span>
                </div>
                <div class="flex items-center">
                  <label
                    class="w-32 text-sm font-medium text-gray-700 dark:text-gray-300"
                    >고객명</label
                  >
                  <input
                    v-model="selectedReservation.customerName"
                    type="text"
                    class="ml-2 px-3 py-1 border border-gray-300 dark:border-gray-600 rounded-md focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white" />
                </div>
                <div class="flex items-center">
                  <label
                    class="w-32 text-sm font-medium text-gray-700 dark:text-gray-300"
                    >청소유형</label
                  >
                  <select
                    v-model="selectedReservation.type"
                    class="ml-2 px-3 py-1 border border-gray-300 dark:border-gray-600 rounded-md focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white">
                    <option value="일반청소">일반청소</option>
                    <option value="입주청소">입주청소</option>
                    <option value="이사청소">이사청소</option>
                  </select>
                </div>
                <div class="flex items-center">
                  <label
                    class="w-32 text-sm font-medium text-gray-700 dark:text-gray-300"
                    >예약일시</label
                  >
                  <input
                    v-model="selectedReservation.date"
                    type="datetime-local"
                    class="ml-2 px-3 py-1 border border-gray-300 dark:border-gray-600 rounded-md focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white" />
                </div>
              </div>
            </div>
          </div>

          <!-- 상태 정보 -->
          <div class="space-y-6">
            <div>
              <h4
                class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-2">
                상태 정보
              </h4>
              <div class="space-y-2">
                <div class="flex items-center">
                  <label
                    class="w-32 text-sm font-medium text-gray-700 dark:text-gray-300"
                    >상태</label
                  >
                  <select
                    v-model="selectedReservation.status"
                    class="ml-2 px-3 py-1 border border-gray-300 dark:border-gray-600 rounded-md focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white">
                    <option value="예약완료">예약완료</option>
                    <option value="진행중">진행중</option>
                    <option value="대기중">대기중</option>
                  </select>
                </div>
                <div class="flex items-center">
                  <label
                    class="w-32 text-sm font-medium text-gray-700 dark:text-gray-300"
                    >담당기사</label
                  >
                  <select
                    v-model="selectedReservation.worker"
                    class="ml-2 px-3 py-1 border border-gray-300 dark:border-gray-600 rounded-md focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white">
                    <option value="-">미배정</option>
                    <option value="이지은">이지은</option>
                    <option value="최윤호">최윤호</option>
                  </select>
                </div>
              </div>
            </div>

            <div>
              <h4
                class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-2">
                메모
              </h4>
              <textarea
                v-model="selectedReservation.memo"
                rows="3"
                class="w-full border border-gray-300 dark:border-gray-600 rounded-md px-3 py-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white dark:bg-gray-700 text-gray-900 dark:text-white"
                placeholder="예약에 대한 메모를 입력하세요"></textarea>
            </div>
          </div>
        </div>
      </div>
      <div
        class="px-6 py-4 bg-gray-50 dark:bg-gray-700 flex justify-end space-x-3">
        <button
          @click="closeModal"
          class="px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-sm font-medium text-gray-700 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-600">
          닫기
        </button>
        <button
          @click="saveReservaton"
          class="px-4 py-2 border border-transparent rounded-md text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700">
          저장
        </button>
      </div>
    </div>
  </div>

 

 
</template>
<script setup>
import Chart from "@/components/Chart.vue";
import DashboardStats from "@/components/DashboardStats.vue";
import SearchTable from "@/components/SearchTable.vue";
import { ref, computed } from "vue";
import Worker_dash from "@/pages/admin/Worker_dash.vue";
import { resersData } from "@/data/resers.js";
// 통계카드 더미
const stats = [
  {
    title: "전체 예약",
    value: "120",
    change: "+12%",
    icon: "fas fa-calendar-check",
    // bgColor: "bg-blue-100 dark:bg-blue-900",
    // textColor: "text-blue-600 dark:text-blue-300",
    bg: "bg-blue-100",
    color: "text-blue-600",
  },
  {
    title: "전체 사용자",
    value: "50",
    change: "+5%",
    icon: "fas fa-users",
    // bgColor: "bg-green-100 dark:bg-green-900",
    // textColor: "text-green-600 dark:text-green-300",
    bg: "bg-green-100",
    color: "text-green-600",
  },
  {
    title: "평균 평점",
    value: "4.8",
    change: "+0.2",
    icon: "fas fa-star",
    // bgColor: "bg-yellow-100 dark:bg-yellow-900",
    // textColor: "text-yellow-600 dark:text-yellow-300",
    bg: "bg-yellow-100",
    color: "text-yellow-600",
  },
];
// 선택된 기사 정보
// const selectedWorker = ref(null);
// 선택된 예약 정보
const selectedReservation = ref(null);
// 서비스유형
const serviceType = ref("all");
// 상태
const receiptStatus = ref("all");
// 예약 정보
const reservations = ref([...resersData]);
// const reservations = ref([
  // {
  //   id: "#1001",
  //   customerName: "김철수",
  //   type: "일반청소",
  //   date: "2025-11-10 14:00",
  //   status: "예약완료",
  //   worker: "이지은",
  // },
  // {
  //   id: "#1002",
  //   customerName: "박영희",
  //   type: "입주청소",
  //   date: "2025-11-11 10:00",
  //   status: "진행중",
  //   worker: "최윤호",
  // },
  // {
  //   id: "#1003",
  //   customerName: "이민수",
  //   type: "이사청소",
  //   date: "2025-11-12 15:00",
  //   status: "대기중",
  //   worker: "-",
  // },
  // {
  //   id: "#1004",
  //   customerName: "정다은",
  //   type: "일반청소",
  //   date: "2025-11-13 11:00",
  //   status: "예약완료",
  //   worker: "이지은",
  // },
  // {
  //   id: "#1005",
  //   customerName: "최준호",
  //   type: "입주청소",
  //   date: "2025-11-14 09:00",
  //   status: "대기중",
  //   worker: "-",
  // },
  // {
  //   id: "#1006",
  //   customerName: "한미영",
  //   type: "이사청소",
  //   date: "2025-11-15 13:00",
  //   status: "예약완료",
  //   worker: "최윤호",
  // },
  // {
  //   id: "#1007",
  //   customerName: "송민준",
  //   type: "일반청소",
  //   date: "2025-11-16 15:00",
  //   status: "진행중",
  //   worker: "이지은",
  // },
  // {
  //   id: "#1008",
  //   customerName: "윤서연",
  //   type: "입주청소",
  //   date: "2025-11-17 10:00",
  //   status: "대기중",
  //   worker: "-",
  // },
// ]);
// 날짜 범위 필터
const dateRange = ref({
  start: "", // 시작일
  end: "", // 종료일
});
// 날짜 문자열에서 YYYY-MM-DD 형식만 추출하는 함수
//  "YYYY-MM-DD" 형식입니다. 시간 문제를 피하기 위해 날짜 문자열만 비교하도록 변경합니다.
const extractDateOnly = (dateString) => {
  // "2025-11-17 10:00" 형식에서 "2025-11-17"만 추출
  // 또는 이미 "2025-11-17" 형식이면 그대로 반환
  if (!dateString) return "";
  return dateString.split(" ")[0].split("T")[0];
};

// 필터링된 예약 목록 계산
const filteredReservaions = computed(() => {
  let result = [...reservations.value]; // 예약목록을 복사
  //    날짜를 필터링:시작일과 종료일을 지정한 경우, 해당 범위내의 예약난 필터링
  // if (dateRange.value.start && dateRange.value.end) {
  //   const startDateStr = extractDateOnly(dateRange.value.start); // "YYYY-MM-DD"
  //   const endDateStr = extractDateOnly(dateRange.value.end); // "YYYY-MM-DD"

  //   result = result.filter((reservation) => {
  //     const reservationDateStr = extractDateOnly(reservation.date); // "YYYY-MM-DD"
  //     // 문자열 비교로 날짜 범위 확인 (종료일 포함)
  //     return (
  //       reservationDateStr >= startDateStr && reservationDateStr <= endDateStr
  //     );
  //   });
  // }
  // 청소서비스 유형 필터링
  // if (serviceType.value !== "all") {
  //   result = result.filter(
  //     (reservation) => reservation.type === serviceType.value
  //   );
  // }

  // // 접수 상태 필터링
  // if (receiptStatus.value !== "all") {
  //   result = result.filter(
  //     (reservation) => reservation.status === receiptStatus.value
  //   );
  // }

  return result;
});
// 페이지네이션 상태
const itemsPerPage = ref(5);

// 테이블 컬럼 정의
const reservationColumns = [
  { label: "예약번호", key: "id" },
  { label: "고객명", key: "customerName" },
  { label: "청소유형", key: "type" },
  {
    label: "예약일시",
    key: "date",

    render: (item) => formatDate(item.date),
  },
  {
    label: "상태",
    key: "status",
    render: (item) =>
      `<span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${getStatusClass(
        item.status
      )}">${item.status}</span>`,
  },
  { label: "담당기사", key: "worker" },
  {
    label: "액션",
    key: "action",
    render: (item) =>
      `<button onclick="window.handleReservationClick('${item.id}')" class="text-indigo-600 dark:text-indigo-400 hover:text-indigo-900 dark:hover:text-indigo-300 mr-3"><i class="fa-solid fa-eye mr-1"></i> 상세</button>`,
  },
];

// 필터 옵션
const dashboardFilterOptions = [
  {
    key: "serviceType",
    options: [
      { value: "all", label: "전체" },
      { value: "일반청소", label: "일반청소" },
      { value: "입주청소", label: "입주청소" },
      { value: "이사청소", label: "이사청소" },
    ],
  },
  {
    key: "receiptStatus",
    options: [
      { value: "all", label: "전체" },
      { value: "예약완료", label: "예약완료" },
      { value: "진행중", label: "진행중" },
      { value: "대기중", label: "대기중" },
    ],
  },
];

// 커스텀 필터 함수 (날짜 범위 포함)
const dashboardFilterFn = (data, filters) => {
  let result = [...data];

  // 날짜 범위 필터링
  if (dateRange.value.start && dateRange.value.end) {
    const startDateStr = extractDateOnly(dateRange.value.start);
    const endDateStr = extractDateOnly(dateRange.value.end);

    result = result.filter((reservation) => {
      const reservationDateStr = extractDateOnly(reservation.date);
      // 문자열 비교로 날짜 범위 확인 (종료일 포함)
      return (
        reservationDateStr >= startDateStr && reservationDateStr <= endDateStr
      );
    });
  }

  // 서비스 유형 필터링
  if (filters.serviceType && filters.serviceType !== "all") {
    result = result.filter(
      (reservation) => reservation.type === filters.serviceType
    );
  }

  // 접수 상태 필터링
  if (filters.receiptStatus && filters.receiptStatus !== "all") {
    result = result.filter(
      (reservation) => reservation.status === filters.receiptStatus
    );
  }

  return result;
};

// 행 클릭 핸들러
const handleRowClick = (item) => {
  showReservationDetails(item);
};

// 전역 함수로 등록 (컴포넌트 내부에서 사용)
window.handleReservationClick = (id) => {
  const reservation = reservations.value.find((r) => r.id === id);
  if (reservation) {
    showReservationDetails(reservation);
  }
};

// 날짜 포맷을 변경하는 함수
// 입력된 날짜 값을 'yyyy년 MM월 dd일 (요일)' 형식으로 변환
const formatDate = (date) => {
  return new Date(date).toLocaleDateString("ko-KR", {
    year: "numeric", // 년도
    month: "long", // 월 (한글 월 이름)
    day: "numeric", // 일
    weekday: "long", // 요일 (한글 요일 이름)
  });
};
// 상태 css
const getStatusClass = (status) => {
  const statusClasses = {
    예약완료:
      "bg-green-100 dark:bg-green-900 text-green-800 dark:text-green-300",
    진행중: "bg-blue-100 dark:bg-blue-900 text-blue-800 dark:text-blue-300",
    대기중: "bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-300",
    확정: "bg-green-100 dark:bg-green-900 text-green-800 dark:text-green-300",
    대기: "bg-yellow-100 dark:bg-yellow-900 text-yellow-800 dark:text-yellow-300",
    취소: "bg-red-100 dark:bg-red-900 text-red-800 dark:text-red-300",
    활동중: "bg-green-100 dark:bg-green-900 text-green-800 dark:text-green-300",
  };
  return (
    statusClasses[status] ||
    "bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-300"
  );
};

// 예약관리 상세 모달
const showReservationDetails = (reservation) => {
  selectedReservation.value = { ...reservation, memo: reservation.memo || "" };
};
// 예약상세 모달 닫기
const closeModal = () => {
  selectedReservation.value = null;
};
// 예약 정보 저장 함수
const saveReservaton = () => {
  // 입력값 유효성 검사
  if (
    !selectedReservation.value.customerName ||
    !selectedReservation.value.date
  ) {
    alert("고객명과 예약일시는 필수 입력 항목입니다.");
    return;
  }
  const index = reservations.value.findIndex(
    (r) => r.id === selectedReservation.value.id
  );
  if (index !== -1) {
    reservations.value[index] = { ...selectedReservation.value };
  }
  //   모달닫기
  closeModal();
};

// 최근예약
const recentReservations = ref([
  { id: 1, customerName: "김철수", date: "2024-03-20", status: "확정" },
  { id: 2, customerName: "이영희", date: "2024-03-21", status: "대기" },
  { id: 3, customerName: "박민수", date: "2024-03-22", status: "취소" },
  { id: 4, customerName: "정지은", date: "2024-03-23", status: "확정" },
]);
</script>
