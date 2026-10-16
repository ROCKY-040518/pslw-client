<template>
  <div class="dashboard-container">
    <h1>팰월드 데디케이티드 서버 매니저</h1>

    <!-- 1. 서버 상태 및 제어 패널 -->
    <section class="control-panel">
      <h2>서버 제어</h2>
      <p>현재 상태: 
        <strong :class="status === 'RUNNING' ? 'text-green' : 'text-red'">
          {{ status === 'RUNNING' ? '🟢 실행 중' : '🔴 정지됨' }}
        </strong>
      </p>
      
      <div class="button-group">
        <button @click="startServer" :disabled="status === 'RUNNING'" class="btn-start">시작</button>
        <button @click="stopServer" :disabled="status !== 'RUNNING'" class="btn-stop">안전 종료</button>
        <button @click="restartServer" class="btn-restart">재시작</button>
      </div>
    </section>

    <hr />

    <!-- 2. 설정 편집 패널 -->
    <section class="settings-panel">
      <h2>서버 설정 (PalWorldSettings.ini)</h2>
      <p class="warning">⚠️ 설정을 적용하려면 변경 후 서버를 재시작해야 합니다.</p>
      
      <form @submit.prevent="saveSettings" class="settings-form">
        <!-- 동적으로 설정 키-값 렌더링 -->
        <div v-for="(value, key) in settings" :key="key" class="form-group">
          <label :for="key">{{ key }}</label>
          <input :id="key" v-model="settings[key]" type="text" />
        </div>
        
        <button type="submit" class="btn-save" :disabled="isSaving">
          {{ isSaving ? '저장 중...' : '설정 저장' }}
        </button>
      </form>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// 백엔드 API 주소 (공유기 DDNS 환경 적용)
const API_BASE = 'http://rocky0518.iptime.org:8080/api'

// 상태 관리
const status = ref('UNKNOWN')
const settings = ref({})
const isSaving = ref(false)

// [API] 서버 상태 조회
const fetchStatus = async () => {
  try {
    const res = await axios.get(`${API_BASE}/server/status`)
    status.value = res.data.status
  } catch (error) {
    console.error('상태 조회 실패:', error)
    status.value = 'ERROR'
  }
}

// [API] 서버 시작
const startServer = async () => {
  try {
    await axios.post(`${API_BASE}/server/start`)
    alert('서버 시작 명령을 전송했습니다.')
    await fetchStatus()
  } catch (error) {
    alert('서버 시작 실패')
  }
}

// [API] 서버 안전 종료
const stopServer = async () => {
  try {
    status.value = 'STOPPING' // UI 즉시 반영
    await axios.post(`${API_BASE}/server/stop`)
    alert('서버가 안전하게 종료되었습니다.')
    await fetchStatus()
  } catch (error) {
    alert('서버 종료 실패')
    await fetchStatus()
  }
}

// [API] 서버 재시작
const restartServer = async () => {
  try {
    status.value = 'RESTARTING'
    await axios.post(`${API_BASE}/server/restart`)
    alert('서버 재시작이 완료되었습니다.')
    await fetchStatus()
  } catch (error) {
    alert('서버 재시작 실패')
    await fetchStatus()
  }
}

// [API] 설정 불러오기
const fetchSettings = async () => {
  try {
    const res = await axios.get(`${API_BASE}/settings`)
    settings.value = res.data
  } catch (error) {
    console.error('설정 불러오기 실패:', error)
  }
}

// [API] 설정 저장 (PATCH)
const saveSettings = async () => {
  if (!confirm('설정을 덮어쓰시겠습니까? (서버 종료 상태 권장)')) return
  
  isSaving.value = true
  try {
    await axios.patch(`${API_BASE}/settings`, settings.value)
    alert('설정이 성공적으로 저장되었습니다.')
  } catch (error) {
    console.error('설정 저장 실패:', error)
    alert('저장에 실패했습니다.')
  } finally {
    isSaving.value = false
  }
}

// 라이프사이클: 컴포넌트 로드 시 초기 데이터 가져오기
onMounted(() => {
  fetchStatus()
  fetchSettings()
})
</script>

<style scoped>
/* 화면 전체 및 패널 디자인 */
.dashboard-container { 
  max-width: 800px; /* 노트북 화면을 위해 너비 확장 */
  margin: 0 auto; 
  padding: 30px 20px; 
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #e0e0e0;
}
h1 {
  line-height: 1.4;
  word-break: keep-all; /* 단어 단위 줄바꿈으로 글자 겹침 완벽 해결 */
  text-align: center;
  margin-bottom: 40px;
  color: #ffffff;
}
.control-panel, .settings-panel {
  background-color: #2a2a2a;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.3);
  margin-bottom: 20px;
}

/* 상단 서버 제어 버튼 */
.button-group { 
  display: flex; 
  gap: 15px; 
  margin-top: 20px; 
}
button { 
  padding: 15px 20px; 
  border: none; 
  border-radius: 8px; 
  cursor: pointer; 
  font-weight: bold; 
  font-size: 1rem;
  transition: all 0.2s; 
}
button:hover:not(:disabled) { filter: brightness(1.15); transform: translateY(-2px); }
button:disabled { opacity: 0.4; cursor: not-allowed; }
.btn-start { background-color: #4CAF50; color: white; flex: 1; }
.btn-stop { background-color: #d32f2f; color: white; flex: 1; }
.btn-restart { background-color: #f57c00; color: white; flex: 1; }

/* 하단 설정 폼 및 텍스트 */
.btn-save { 
  background-color: #2196F3; 
  color: white; 
  width: 100%; 
  padding: 18px; 
  margin-top: 30px; 
  font-size: 1.2rem; 
}
.warning { color: #ffb74d; font-size: 0.95em; margin-bottom: 20px; font-weight: bold; }
.settings-form .form-group { margin-bottom: 18px; display: flex; flex-direction: column; }
.settings-form label { font-size: 0.9em; margin-bottom: 8px; color: #aaaaaa; }
.settings-form input { 
  padding: 12px; 
  border: 1px solid #444; 
  border-radius: 6px; 
  background-color: #1e1e1e; 
  color: #ffffff; 
  font-size: 1rem;
}
.settings-form input:focus { border-color: #2196F3; outline: none; }
.text-green { color: #4CAF50; }
.text-red { color: #d32f2f; }
hr { border: none; }
</style>