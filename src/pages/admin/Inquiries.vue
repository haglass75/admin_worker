<template>
  <div class="inquiries-page">
    <h2 class="text-3xl font-bold mb-6">문의 관리</h2>

    <!-- 필터 섹션 -->
    <div class="bg-white rounded-lg shadow-sm p-4 mb-6">
      <div class="flex flex-wrap gap-2">
        <button
          v-for="filter in filters"
          :key="filter"
          @click="currentFilter = filter"
          :class="[
            'px-4 py-2 rounded-lg text-sm font-medium transition-colors',
            currentFilter === filter
              ? 'bg-indigo-600 text-white'
              : 'bg-gray-100 text-gray-700 hover:bg-gray-200',
          ]">
          {{ filter }}
        </button>
      </div>
    </div>

    <!-- 검색 및 통계 -->
    <DashboardStats :stats="stats" class="md:grid-cols-4" />

    <!-- 문의 목록 (공통 컴포넌트 사용) -->
    <SearchTable
      :data="inquiries"
      :columns="inquiryColumns"
      search-placeholder="고객명, 제목, 내용으로 검색..."
      :search-fields="['title', 'content', 'customerName']"
      :filter-options="inquiryFilterOptions"
      :filter-fn="inquiryFilterFn"
      :items-per-page="itemsPerPage"
      table-title=""
      total-label="개"
      @row-click="handleInquiryRowClick" />

    <!-- 문의 상세 모달 -->
    <div
      v-if="selectedInquiry"
      class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center p-4">
      <div
        class="bg-white rounded-lg max-w-3xl w-full max-h-[90vh] overflow-y-auto">
        <div class="p-6">
          <div class="flex justify-between items-start mb-4">
            <h3 class="text-2xl font-bold">문의 상세</h3>
            <button
              @click="selectedInquiry = null"
              class="text-gray-400 hover:text-gray-600">
              <i class="fas fa-times text-xl"></i>
            </button>
          </div>

          <div class="space-y-4">
            <div class="border-b pb-4">
              <div class="flex items-center gap-2 mb-2">
                <span
                  :class="[
                    'px-2 py-1 text-xs font-medium rounded-full',
                    getCategoryClass(selectedInquiry.category),
                  ]">
                  {{ selectedInquiry.category }}
                </span>
                <span
                  :class="[
                    'px-2 py-1 text-xs font-medium rounded-full',
                    selectedInquiry.answered
                      ? 'bg-green-100 text-green-800'
                      : 'bg-yellow-100 text-yellow-800',
                  ]">
                  {{ selectedInquiry.answered ? "답변완료" : "답변대기" }}
                </span>
              </div>
              <h4 class="text-xl font-semibold mb-2">
                {{ selectedInquiry.title }}
              </h4>
              <div class="text-sm text-gray-600">
                <span class="mr-4"
                  >고객명: {{ selectedInquiry.customerName }}</span
                >
                <span>등록일: {{ formatDate(selectedInquiry.date) }}</span>
              </div>
            </div>

            <div class="border-b pb-4">
              <h5 class="font-semibold mb-2">문의 내용</h5>
              <p class="text-gray-700 whitespace-pre-wrap">
                {{ selectedInquiry.content }}
              </p>
            </div>

            <div v-if="selectedInquiry.answer" class="border-b pb-4">
              <h5 class="font-semibold mb-2 text-green-700">답변 내용</h5>
              <p class="text-gray-700 whitespace-pre-wrap">
                {{ selectedInquiry.answer }}
              </p>
              <p class="text-xs text-gray-500 mt-2">
                답변일: {{ formatDate(selectedInquiry.answeredDate) }}
              </p>
            </div>

            <div
              v-if="!selectedInquiry.answered"
              class="flex justify-end gap-3">
              <button
                @click="selectedInquiry = null"
                class="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors">
                닫기
              </button>
              <button
                @click="replyToInquiry(selectedInquiry)"
                class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors">
                <i class="fas fa-reply mr-2"></i>
                답변 작성
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 답변 작성 모달 -->
    <div
      v-if="replyingToInquiry"
      class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center p-4">
      <div class="bg-white rounded-lg max-w-2xl w-full">
        <div class="p-6">
          <div class="flex justify-between items-start mb-4">
            <h3 class="text-2xl font-bold">답변 작성</h3>
            <button
              @click="replyingToInquiry = null"
              class="text-gray-400 hover:text-gray-600">
              <i class="fas fa-times text-xl"></i>
            </button>
          </div>

          <div class="space-y-4">
            <div class="border-b pb-4">
              <p class="text-sm text-gray-600 mb-2">문의 제목</p>
              <p class="font-semibold">{{ replyingToInquiry.title }}</p>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-700 mb-2"
                >답변 내용</label
              >
              <textarea
                v-model="replyContent"
                rows="8"
                placeholder="답변 내용을 입력하세요..."
                class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"></textarea>
            </div>

            <div class="flex justify-end gap-3">
              <button
                @click="replyingToInquiry = null"
                class="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors">
                취소
              </button>
              <button
                @click="submitReply"
                class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors">
                <i class="fas fa-check mr-2"></i>
                답변 등록
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import DashboardStats from "@/components/DashboardStats.vue";
import SearchTable from "@/components/SearchTable.vue";
import { ref, computed } from "vue";

const filters = ["전체", "답변대기", "답변완료"];
const currentFilter = ref("전체");
const itemsPerPage = ref(10);
const selectedInquiry = ref(null);
const replyingToInquiry = ref(null);
const replyContent = ref("");

// 테이블 컬럼 정의
const inquiryColumns = [
  { label: "번호", key: "id" },
  {
    label: "카테고리",
    key: "category",
    render: (item) =>
      `<span class="px-2 py-1 text-xs font-medium rounded-full ${getCategoryClass(
        item.category
      )}">${item.category}</span>`,
  },
  {
    label: "제목",
    key: "title",
    render: (item) => {
      const attachmentIcon = item.hasAttachment
        ? '<i class="fas fa-paperclip ml-2 text-gray-400 text-xs"></i>'
        : "";
      return `<div class="flex items-center"><span>${item.title}</span>${attachmentIcon}</div>`;
    },
  },
  { label: "고객명", key: "customerName" },
  {
    label: "등록일",
    key: "date",
    render: (item) => formatDate(item.date),
  },
  {
    label: "상태",
    key: "answered",
    render: (item) => {
      const statusClass = item.answered
        ? "bg-green-100 text-green-800"
        : "bg-yellow-100 text-yellow-800";
      const statusText = item.answered ? "답변완료" : "답변대기";
      return `<span class="px-2 py-1 text-xs font-medium rounded-full ${statusClass}">${statusText}</span>`;
    },
  },
  {
    label: "관리",
    key: "action",
    render: (item) => {
      const viewBtn = `<button onclick="window.handleInquiryView('${item.id}')" class="text-indigo-600 hover:text-indigo-900 mr-3"><i class="fas fa-eye"></i></button>`;
      const replyBtn = !item.answered
        ? `<button onclick="window.handleInquiryReply('${item.id}')" class="text-green-600 hover:text-green-900"><i class="fas fa-reply"></i></button>`
        : "";
      return viewBtn + replyBtn;
    },
  },
];

// 필터 옵션
const inquiryFilterOptions = [
  {
    key: "statusFilter",
    options: [
      { value: "전체", label: "전체" },
      { value: "답변대기", label: "답변대기" },
      { value: "답변완료", label: "답변완료" },
    ],
  },
];

// 커스텀 필터 함수 (상단 필터와 통합)
const inquiryFilterFn = (data, filters) => {
  let result = [...data];

  // 상단 필터 버튼의 값 사용 (currentFilter)
  if (currentFilter.value === "답변대기") {
    result = result.filter((i) => !i.answered);
  } else if (currentFilter.value === "답변완료") {
    result = result.filter((i) => i.answered);
  }

  // SearchTable의 필터 옵션 값도 처리
  if (filters.statusFilter && filters.statusFilter !== "전체") {
    if (filters.statusFilter === "답변대기") {
      result = result.filter((i) => !i.answered);
    } else if (filters.statusFilter === "답변완료") {
      result = result.filter((i) => i.answered);
    }
  }

  return result;
};

// 행 클릭 핸들러
const handleInquiryRowClick = (item) => {
  viewInquiry(item);
};

// 전역 함수 등록
window.handleInquiryView = (id) => {
  const inquiry = inquiries.value.find((i) => i.id === parseInt(id));
  if (inquiry) {
    viewInquiry(inquiry);
  }
};

window.handleInquiryReply = (id) => {
  const inquiry = inquiries.value.find((i) => i.id === parseInt(id));
  if (inquiry) {
    replyToInquiry(inquiry);
  }
};
// 통계더미
const stats = [
  {
    title: "전체 문의",
    value: 5,
    icon: "fas fa-inbox",
    bg: "bg-blue-100",
    color: "text-blue-600",
  },
  {
    title: "답변 대기",
    value: 3,
    icon: "fas fa-clock",
    bg: "bg-yellow-100",
    color: "text-yellow-600",
  },
  {
    title: "답변 완료",
    value: 2,
    icon: "fas fa-check-circle",
    bg: "bg-green-100",
    color: "text-green-600",
  },
  {
    title: "오늘 문의",
    value: 0,
    icon: "fas fa-comments",
    bg: "bg-purple-100",
    color: "text-purple-600",
  },
];
// 샘플 데이터
const inquiries = ref([
  {
    id: 1001,
    category: "서비스 문의",
    title: "청소 서비스 이용 방법이 궁금합니다",
    customerName: "김철수",
    date: new Date("2024-01-15"),
    answered: false,
    content: "청소 서비스를 처음 이용하는데 어떤 절차로 진행되나요?",
    hasAttachment: false,
  },
  {
    id: 1002,
    category: "결제 문의",
    title: "결제 방법 변경이 가능한가요?",
    customerName: "이영희",
    date: new Date("2024-01-14"),
    answered: true,
    content: "신용카드에서 계좌이체로 변경하고 싶습니다.",
    answer:
      "네, 가능합니다. 결제 방법은 마이페이지에서 변경 가능하며, 다음 결제부터 적용됩니다.",
    answeredDate: new Date("2024-01-14"),
    hasAttachment: true,
  },
  {
    id: 1003,
    category: "불만 접수",
    title: "작업 품질이 만족스럽지 않습니다",
    customerName: "박민수",
    date: new Date("2024-01-13"),
    answered: false,
    content: "지난번 청소가 제대로 안 된 것 같습니다. 재청소 가능한가요?",
    hasAttachment: false,
  },
  {
    id: 1004,
    category: "예약 변경",
    title: "예약 시간 변경하고 싶습니다",
    customerName: "정수진",
    date: new Date("2024-01-12"),
    answered: true,
    content: "1월 20일 오후 2시로 예약을 변경 가능한가요?",
    answer: "네, 가능합니다. 예약 시간이 변경되었습니다.",
    answeredDate: new Date("2024-01-12"),
    hasAttachment: false,
  },
  {
    id: 1005,
    category: "기타",
    title: "면세 청소 가능한가요?",
    customerName: "최지영",
    date: new Date("2024-01-11"),
    answered: false,
    content: "특수 청소가 필요한 상황입니다.",
    hasAttachment: true,
  },
]);

// 기존 필터링 코드는 SearchTable 컴포넌트로 이동됨

// 메서드
function getCategoryClass(category) {
  const classes = {
    "서비스 문의": "bg-blue-100 text-blue-800",
    "결제 문의": "bg-green-100 text-green-800",
    "불만 접수": "bg-red-100 text-red-800",
    "예약 변경": "bg-yellow-100 text-yellow-800",
    기타: "bg-gray-100 text-gray-800",
  };
  return classes[category] || "bg-gray-100 text-gray-800";
}

function formatDate(date) {
  if (!date) return "";
  return new Date(date).toLocaleDateString("ko-KR", {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  });
}

function viewInquiry(inquiry) {
  selectedInquiry.value = inquiry;
}

function replyToInquiry(inquiry) {
  replyingToInquiry.value = inquiry;
  replyContent.value = "";
}

function submitReply() {
  if (!replyContent.value.trim()) {
    alert("답변 내용을 입력해주세요.");
    return;
  }

  const inquiryIndex = inquiries.value.findIndex(
    (i) => i.id === replyingToInquiry.value.id
  );
  if (inquiryIndex !== -1) {
    inquiries.value[inquiryIndex].answered = true;
    inquiries.value[inquiryIndex].answer = replyContent.value;
    inquiries.value[inquiryIndex].answeredDate = new Date();
  }

  replyingToInquiry.value = null;
  replyContent.value = "";
  selectedInquiry.value = null;
  alert("답변이 등록되었습니다.");
}
</script>

<style scoped>
.inquiries-page {
  max-width: 100%;
}
</style>
