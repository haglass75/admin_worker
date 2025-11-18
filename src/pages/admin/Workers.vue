<template>
  <div
    class="p-4 rounded space-y-6 text-gray-800 dark:bg-gray-900 dark:text-gray-200">
    <!-- 페이지 헤더 -->
    <div class="flex justify-between items-center">
      <div>
        <h1 class="text-2xl font-bold text-gray-800 dark:text-white">
          기사 관리
        </h1>
        <p class="text-sm text-gray-600 dark:text-gray-400 mt-1">
          기사 정보를 관리하고 상태를 확인할 수 있습니다.
        </p>
      </div>
    </div>

    <!-- 통계 카드 -->

    <DashboardStats :stats="stats" />

    <!-- 기사 목록 (공통 컴포넌트 사용) -->
    <Worker_dash />
  </div>
</template>
<script setup>
import DashboardStats from "@/components/DashboardStats.vue";
import SearchTable from "@/components/SearchTable.vue";
import { ref, computed } from "vue";
import Worker_dash from "@/pages/admin/Worker_dash.vue";
import { workersData } from "@/data/workers.js";

// 기사목록 - 외부 파일에서 가져오기
const workers = ref([...workersData]);

// 통계를 computed로 만들어서 workers가 변경될 때 자동으로 업데이트되도록 함
const stats = computed(() => [
  {
    title: "전체 기사",
    value: `${workers.value.length}명`,
    change: "+3명",
    icon: "fas fa-user-tie",
    bg: "bg-blue-100 dark:bg-blue-900",
    color: "text-blue-600 dark:text-blue-300",
  },
  {
    title: "활동중",
    value: `${workers.value.filter((w) => w.status === "활동중").length}명`,
    change: "+1명",
    icon: "fas fa-check-circle",
    bg: "bg-green-100 dark:bg-green-900",
    color: "text-green-600 dark:text-green-300",
  },
  {
    title: "평균 평점",
    value:
      workers.value.length > 0
        ? (
            workers.value.reduce((sum, w) => sum + w.rating, 0) /
            workers.value.length
          ).toFixed(1)
        : "0.0",
    change: "+0.1",
    icon: "fas fa-star",
    bg: "bg-yellow-100 dark:bg-yellow-900",
    color: "text-yellow-600 dark:text-yellow-300",
  },
]);
</script>
