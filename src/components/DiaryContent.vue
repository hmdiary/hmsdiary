<script setup>
import { computed } from 'vue';
import { marked } from 'marked';

const props = defineProps({
    diary: { type: Object, default: null },
});
defineEmits(['back']);

// 配置 marked
marked.setOptions({
    breaks: true, // 回车转为 <br>
    gfm: true,    // 启动 GitHub Flavored Markdown
});

const diaryHtml = computed(() => {
    if (props.diary && props.diary.content) {
        let content = props.diary.content;
        // 处理图片路径：将相对路径转换为正确的路径
        content = content.replace(/!\[(.*?)\]\(\.\.\/pics\/(.*?)\)/g, '![$1](./pics/$2)');
        return marked(content);
    }
    return '';
});

const formattedDate = computed(() => {
    if (!props.diary) return '';
    return new Date(props.diary.date).toLocaleDateString('zh-CN', {
        year: 'numeric', month: 'long', day: 'numeric', weekday: 'long'
    });
});
</script>

<template>
    <div class="diary-content-container">
        <div v-if="!diary" class="welcome-screen">
            <h2>山居札记</h2>
            <p>选择一篇日记开始阅读</p>
        </div>

        <template v-else>
            <header class="diary-header">
                <button @click="$emit('back')" class="back-button">← 返回列表</button>
                <h1>{{ diary.title }}</h1>
                <div class="meta">{{ formattedDate }}</div>
            </header>
            <div class="content-scroll-area">
                <div class="diary-body" v-html="diaryHtml"></div>
            </div>
        </template>
    </div>
</template>

<style scoped>
/* 核心布局: flex让内容区撑满 */
.diary-content-container {
    height: 100%;
    width: 100%;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    /* 防止容器自身出现滚动条 */
}

.welcome-screen {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #aaa;
}

.welcome-screen h2 {
    color: #2f4f2f;
    font-weight: normal;
}

.diary-header {
    padding: 15px;
    border-bottom: 1px solid #e0e0e0;
    flex-shrink: 0;
    position: relative;
    text-align: center;
}

.back-button {
    display: none;
    /* 默认隐藏，仅在移动端显示 */
    position: absolute;
    top: 50%;
    left: 15px;
    transform: translateY(-50%);
    background: #f0f0f0;
    border: 1px solid #ccc;
    padding: 6px 12px;
    border-radius: 6px;
    cursor: pointer;
}

.diary-header h1 {
    font-size: 22px;
    font-weight: normal;
}

.diary-header .meta {
    font-size: 13px;
    color: #888;
    margin-top: 8px;
}

.content-scroll-area {
    flex-grow: 1;
    /* 关键：占据剩余空间 */
    overflow-y: auto;
    overflow-x: hidden;
    /* 防止水平滚动 */
    padding: 20px;
    height: 0;
    /* 关键：设置高度为0，让flex-grow控制实际高度 */
}

.diary-body {
    max-width: 680px;
    margin: 0 auto;
    line-height: 1.8;
}

/* Markdown 渲染样式 */
.diary-body:deep(h1),
.diary-body:deep(h2),
.diary-body:deep(h3) {
    margin-top: 1.5em;
    margin-bottom: 0.5em;
    font-weight: normal;
}

.diary-body:deep(p) {
    margin-bottom: 1em;
}

.diary-body:deep(a) {
    color: #556b2f;
}

.diary-body:deep(blockquote) {
    margin: 1em 0;
    padding: 0.5em 1em;
    border-left: 3px solid #ccc;
    background-color: #f8f8f8;
}

.diary-body:deep(img) {
    max-width: 100%;
    border-radius: 4px;
}

/* 更多 markdown 样式可按需添加 */

/* 移动端样式 */
@media (max-width: 768px) {
    .back-button {
        display: block;
        /* 在移动端显示返回按钮 */
    }
}

/* 桌面端样式 */
@media (min-width: 769px) {
    .diary-header {
        padding: 30px;
    }
}
</style>