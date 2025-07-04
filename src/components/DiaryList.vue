<script setup>
defineProps({
    diaries: { type: Array, required: true },
    selectedDiaryId: { type: String, default: null },
});
defineEmits(['select-diary']);

const formatDate = (dateStr) => {
    const date = new Date(dateStr);
    const today = new Date();
    const diffDays = Math.floor((today - date) / (1000 * 60 * 60 * 24));
    if (diffDays === 0) return '今日';
    if (diffDays === 1) return '昨日';
    return date.toLocaleDateString('zh-CN', { month: 'long', day: 'numeric' });
};
</script>

<template>
    <div class="diary-list-container">
        <div class="list-header">
            <h3>时光记录</h3>
            <div class="count">{{ diaries.length }} 篇</div>
        </div>

        <div v-if="diaries.length === 0" class="empty-state">
            <p>暂无记录</p>
        </div>

        <div v-else class="scroll-area">
            <div v-for="diary in diaries" :key="diary.id" class="diary-item"
                :class="{ active: selectedDiaryId === diary.id }" @click="$emit('select-diary', diary.id)">
                <div class="date">{{ formatDate(diary.date) }}</div>
                <div class="title">{{ diary.title }}</div>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* 核心布局: flex让滚动区撑满剩余空间 */
.diary-list-container {
    display: flex;
    flex-direction: column;
    height: 100%;
    width: 100%;
    background-color: #f9f9f9;
}

.list-header {
    padding: 15px;
    border-bottom: 1px solid #e0e0e0;
    flex-shrink: 0;
}

.list-header h3 {
    font-size: 16px;
    margin-bottom: 2px;
}

.list-header .count {
    font-size: 12px;
    color: #888;
}

.scroll-area {
    flex-grow: 1;
    /* 关键：让此区域占据所有剩余空间 */
    overflow-y: auto;
    /* 内容超出时滚动 */
    padding: 10px;
}

.empty-state {
    flex-grow: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #888;
}

.diary-item {
    padding: 15px;
    margin-bottom: 8px;
    border-radius: 8px;
    cursor: pointer;
    background-color: #fff;
    border: 1px solid #f0f0f0;
    transition: all 0.2s ease;
}

.diary-item:hover {
    border-color: #ccc;
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.05);
}

.diary-item.active {
    border-color: #556b2f;
    background-color: #e8f5e8;
}

.date {
    font-size: 12px;
    color: #888;
    margin-bottom: 5px;
}

.title {
    font-weight: 500;
    color: #2f4f2f;
}

/* 桌面端样式微调 */
@media (min-width: 769px) {
    .diary-list-container {
        border-right: 1px solid #ddd;
    }

    .scroll-area {
        padding: 10px 0;
    }

    .diary-item {
        border-radius: 0;
        margin-bottom: 0;
        border: none;
        border-bottom: 1px solid #f0f0f0;
        padding: 15px 20px;
    }

    .diary-item:hover {
        transform: none;
        box-shadow: none;
        background-color: #f0f0f0;
    }
}
</style>