<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import DiaryList from './components/DiaryList.vue';
import DiaryContent from './components/DiaryContent.vue';

// --- State ---
const diaries = ref([]);
const selectedDiaryId = ref(null);
const isMobile = ref(window.innerWidth <= 768);

// --- Computed ---
const selectedDiary = computed(() => {
  return diaries.value.find(d => d.id === selectedDiaryId.value) || null;
});

// --- Logic ---
const loadDiaries = async () => {
  const diaryModules = import.meta.glob('./diaries/*.md', { as: 'raw', eager: true });
  const loadedDiaries = [];
  for (const path in diaryModules) {
    const content = diaryModules[path];
    const filename = path.split('/').pop();
    const match = filename.match(/^(\d{4}-\d{2}-\d{2})-(.+)\.md$/);
    if (match) {
      const [, date, titleSlug] = match;
      loadedDiaries.push({
        id: filename,
        title: titleSlug.replace(/-/g, ' '),
        date,
        content,
        dateObj: new Date(date)
      });
    }
  }
  diaries.value = loadedDiaries.sort((a, b) => b.dateObj - a.dateObj);
};

const handleSelectDiary = (diaryId) => {
  selectedDiaryId.value = diaryId;
};

const handleBackToList = () => {
  selectedDiaryId.value = null;
};

// --- Lifecycle ---
const checkMobile = () => {
  isMobile.value = window.innerWidth <= 768;
  // 如果从移动端切换回桌面端，且没有选中日记，则默认选中第一篇
  if (!isMobile.value && !selectedDiaryId.value && diaries.value.length > 0) {
    selectedDiaryId.value = diaries.value[0].id;
  }
};

onMounted(() => {
  loadDiaries();
  // 仅在桌面端自动选择第一篇
  if (!isMobile.value && diaries.value.length > 0) {
    selectedDiaryId.value = diaries.value[0].id;
  }
  window.addEventListener('resize', checkMobile);
});

onUnmounted(() => {
  window.removeEventListener('resize', checkMobile);
});
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>山居札记</h1>
      <p>远山如黛，心如止水</p>
    </header>

    <main class="app-content">
      <template v-if="isMobile">
        <DiaryList v-if="!selectedDiaryId" :diaries="diaries" @select-diary="handleSelectDiary" />
        <DiaryContent v-else :diary="selectedDiary" @back="handleBackToList" />
      </template>

      <template v-else>
        <DiaryList :diaries="diaries" :selected-diary-id="selectedDiaryId" @select-diary="handleSelectDiary" />
        <DiaryContent :diary="selectedDiary" />
      </template>
    </main>
  </div>
</template>

<style>
/* --- 全局与重置 --- */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Times New Roman', 'SimSun', serif;
  background-color: #f5f5dc;
  color: #2f4f2f;
}

/* --- 主要布局 --- */
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  /* 防止body出现滚动条 */
}

.app-header {
  padding: 15px;
  text-align: center;
  background-color: #2f4f2f;
  color: #f5f5dc;
  border-bottom: 3px solid #556b2f;
  flex-shrink: 0;
  /* 防止头部被压缩 */
  z-index: 10;
}

.app-header h1 {
  font-size: 20px;
  font-weight: normal;
  letter-spacing: 2px;
}

.app-header p {
  font-size: 12px;
  color: #d2b48c;
  opacity: 0.9;
}

.app-content {
  flex-grow: 1;
  /* 占据所有剩余垂直空间 */
  background-color: #fefefe;
  overflow: hidden;
  /* 强制内容区管理自己的滚动 */
  position: relative;
}

/* --- 桌面端布局 (> 768px) --- */
@media (min-width: 769px) {
  .app-content {
    display: grid;
    grid-template-columns: 280px 1fr;
    /* 列表固定宽度，内容自适应 */
  }

  .app-header {
    padding: 25px;
  }

  .app-header h1 {
    font-size: 28px;
  }

  .app-header p {
    font-size: 14px;
  }
}
</style>