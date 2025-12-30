<template>
  <div class="container">
    <div class="header">
      <h1 class="title">🎯 모의면접 질문</h1>
      <p class="subtitle">카테고리별로 면접 질문을 확인하고 답변을 준비하세요</p>
    </div>

    <!-- 카테고리 화면 -->
    <div v-if="view === 'categories'">
      <div class="filter-section">
        <label class="filter-label">난이도 필터</label>
        <select v-model="level" class="select-box">
          <option value="">전체</option>
          <option v-for="l in levels" :key="l" :value="l">{{ l }}</option>
        </select>
      </div>

      <div class="category-grid">
        <button
            v-for="c in categories"
            :key="c"
            @click="openCategory(c)"
            class="category-card"
        >
          <div class="category-icon">📁</div>
          <div class="category-name">{{ c }}</div>
          <div class="category-count">{{ countByCategory(c) }}개 질문</div>
        </button>
      </div>
    </div>

    <!-- 질문 목록 화면 -->
    <div v-else>
      <div class="question-header">
        <button @click="back" class="back-button">
          <span class="arrow">←</span> 뒤로가기
        </button>
        <div class="selected-category">
          <span class="category-icon-small">📁</span>
          {{ selectedCategory }}
        </div>
        <select v-model="level" class="select-box-small">
          <option value="">전체</option>
          <option v-for="l in levels" :key="l" :value="l">{{ l }}</option>
        </select>
      </div>

      <div v-if="filteredQuestions.length === 0" class="empty-state">
        <div class="empty-icon">🔍</div>
        <p>조건에 맞는 질문이 없습니다.</p>
      </div>

      <div v-else class="questions-list">
        <div
            v-for="x in filteredQuestions"
            :key="x.id"
            class="question-card"
            :class="{ 'expanded': opened[x.id] }"
        >
          <div class="question-header-row">
            <div class="question-content">
              <span class="level-badge" :class="getLevelClass(x.level)">{{ x.level }}</span>
              <span class="question-text">{{ x.q }}</span>
            </div>
            <button @click="toggle(x.id)" class="toggle-button">
              {{ opened[x.id] ? "숨기기" : "답변 보기" }}
            </button>
          </div>

          <transition name="slide-fade">
            <div v-if="opened[x.id]" class="answer-section">
              <div class="answer-label">💡 모범 답변</div>
              <div class="answer-text">{{ x.a }}</div>
            </div>
          </transition>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, reactive, ref } from "vue";

const all = ref([]);
const view = ref("categories");
const selectedCategory = ref("");
const level = ref("");
const opened = reactive({});

onMounted(async () => {
  const r = await fetch("/questions.json");
  all.value = await r.json();
});

const categories = computed(() => [...new Set(all.value.map(x => x.category))]);
const levels = computed(() => [...new Set(all.value.map(x => x.level))]);

function countByCategory(cat) {
  return all.value.filter(x => x.category === cat && (!level.value || x.level === level.value)).length;
}

function openCategory(cat) {
  selectedCategory.value = cat;
  view.value = "questions";
  clearOpened();
}

function back() {
  view.value = "categories";
  selectedCategory.value = "";
  clearOpened();
}

function clearOpened() {
  Object.keys(opened).forEach(k => delete opened[k]);
}

const filteredQuestions = computed(() => {
  return all.value.filter(x =>
      x.category === selectedCategory.value &&
      (!level.value || x.level === level.value)
  );
});

function toggle(id) {
  opened[id] = !opened[id];
}

function getLevelClass(level) {
  const map = {
    '하': 'easy',
    '중': 'medium',
    '상': 'hard'
  };
  return map[level] || 'medium';
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.container {
  max-width: 1100px;
  margin: 0 auto;
  padding: 24px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.header {
  text-align: center;
  margin-bottom: 40px;
  padding: 20px;
}

.title {
  font-size: 36px;
  font-weight: 800;
  color: white;
  margin: 0 0 8px 0;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}

.subtitle {
  font-size: 16px;
  color: rgba(255,255,255,0.9);
  margin: 0;
}

.filter-section {
  background: white;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 24px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  display: flex;
  align-items: center;
  gap: 12px;
}

.filter-label {
  font-weight: 600;
  color: #333;
  font-size: 14px;
}

.select-box {
  padding: 10px 16px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 14px;
  background: white;
  cursor: pointer;
  transition: all 0.2s;
  outline: none;
}

.select-box:hover {
  border-color: #667eea;
}

.select-box:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.category-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 16px;
}

.category-card {
  background: white;
  border: none;
  border-radius: 16px;
  padding: 28px 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  text-align: center;
}

.category-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}

.category-icon {
  font-size: 40px;
  margin-bottom: 12px;
}

.category-name {
  font-size: 18px;
  font-weight: 700;
  color: #2d3748;
  margin-bottom: 8px;
}

.category-count {
  font-size: 14px;
  color: #718096;
}

.question-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 24px;
  background: white;
  padding: 16px 20px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.back-button {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 18px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.back-button:hover {
  background: #5568d3;
  transform: translateX(-2px);
}

.arrow {
  font-size: 16px;
}

.selected-category {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 18px;
  font-weight: 700;
  color: #2d3748;
  flex: 1;
}

.category-icon-small {
  font-size: 20px;
}

.select-box-small {
  padding: 8px 14px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 14px;
  background: white;
  cursor: pointer;
  transition: all 0.2s;
  outline: none;
}

.select-box-small:hover {
  border-color: #667eea;
}

.empty-state {
  background: white;
  padding: 60px 20px;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.empty-icon {
  font-size: 60px;
  margin-bottom: 16px;
}

.empty-state p {
  font-size: 16px;
  color: #718096;
  margin: 0;
}

.questions-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.question-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: all 0.3s ease;
}

.question-card:hover {
  box-shadow: 0 4px 16px rgba(0,0,0,0.15);
}

.question-card.expanded {
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.2);
}

.question-header-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
}

.question-content {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  flex: 1;
}

.level-badge {
  display: inline-block;
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 700;
  flex-shrink: 0;
}

.level-badge.easy {
  background: #d4edda;
  color: #155724;
}

.level-badge.medium {
  background: #fff3cd;
  color: #856404;
}

.level-badge.hard {
  background: #f8d7da;
  color: #721c24;
}

.question-text {
  font-size: 16px;
  line-height: 1.6;
  color: #2d3748;
  font-weight: 500;
}

.toggle-button {
  padding: 8px 16px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
  flex-shrink: 0;
}

.toggle-button:hover {
  background: #5568d3;
}

.answer-section {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 2px solid #e2e8f0;
}

.answer-label {
  font-size: 14px;
  font-weight: 700;
  color: #667eea;
  margin-bottom: 12px;
}

.answer-text {
  font-size: 15px;
  line-height: 1.8;
  color: #4a5568;
  background: #f7fafc;
  padding: 16px;
  border-radius: 8px;
  border-left: 4px solid #667eea;
}

.slide-fade-enter-active {
  transition: all 0.3s ease;
}

.slide-fade-leave-active {
  transition: all 0.2s ease;
}

.slide-fade-enter-from {
  transform: translateY(-10px);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}

@media (max-width: 768px) {
  .container {
    padding: 16px;
  }

  .title {
    font-size: 28px;
  }

  .category-grid {
    grid-template-columns: 1fr;
  }

  .question-header {
    flex-wrap: wrap;
  }

  .question-header-row {
    flex-direction: column;
    align-items: stretch;
  }

  .toggle-button {
    width: 100%;
  }
}
</style>