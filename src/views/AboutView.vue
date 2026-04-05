<template>
  <section class="request-page">
    <div class="request-hero">
      <div>
        <p class="hero-kicker">Curated Request Archive</p>
        <h2>ภาพรวมการเบิกอุปกรณ์ในมุมมองที่คม ชัด และอ่านง่าย</h2>
        <p class="hero-text">
          ดึงข้อมูลจาก n8n webhook แล้วสรุปออกมาเป็นแดชบอร์ดสไตล์พรีเมียม ช่วยให้เห็นภาพรวมการใช้งาน
          อุปกรณ์ได้ภายในหน้าเดียว
        </p>
      </div>

      <div class="hero-summary">
        <div class="summary-card">
          <span>รายการทั้งหมด</span>
          <strong>{{ requests.length }}</strong>
        </div>
        <div class="summary-card">
          <span>สถานะ</span>
          <strong>{{ loading ? 'กำลังโหลด' : 'เชื่อมต่อแล้ว' }}</strong>
        </div>
      </div>
    </div>

    <section class="dashboard-shell">
      <div class="toolbar">
        <div>
          <p class="hero-kicker toolbar-kicker">Live Collection</p>
          <h3>ตารางรายการเบิกอุปกรณ์</h3>
          <p class="toolbar-text">รีเฟรชข้อมูลล่าสุดจาก webhook ได้ทันทีโดยไม่ต้องออกจากหน้า</p>
        </div>

        <button class="refresh-btn" @click="fetchData" :disabled="loading">
          <span v-if="loading" class="spinner-border spinner-border-sm"></span>
          <span>{{ loading ? 'กำลังโหลด...' : 'รีเฟรชข้อมูล' }}</span>
        </button>
      </div>

      <div v-if="errorMessage" class="notice notice-error">
        {{ errorMessage }}
      </div>

      <div v-if="loading" class="state-panel">
        <div class="spinner-border"></div>
        <p>กำลังดึงข้อมูลรายการเบิกอุปกรณ์...</p>
      </div>

      <div v-else-if="requests.length" class="table-wrap">
        <table class="request-table">
          <thead>
            <tr>
              <th>#</th>
              <th>เวลา</th>
              <th>ชื่อผู้เบิก</th>
              <th>แผนก</th>
              <th>รายการอุปกรณ์</th>
              <th>จำนวน</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in requests" :key="`${item.timestamp}-${index}`">
              <td>{{ index + 1 }}</td>
              <td>{{ item.timestamp }}</td>
              <td class="name-cell">{{ item.fullname }}</td>
              <td><span class="badge badge-department">{{ item.department }}</span></td>
              <td>{{ item.itemName }}</td>
              <td><span class="badge badge-quantity">{{ item.quantity }}</span></td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-else class="state-panel empty-panel">
        <div class="empty-mark">01</div>
        <h4>ยังไม่มีข้อมูลการเบิก</h4>
        <p>เมื่อมีรายการจากแบบฟอร์ม ระบบจะแสดงข้อมูลในหน้านี้โดยอัตโนมัติ</p>
      </div>
    </section>
  </section>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const requests = ref([])
const loading = ref(false)
const errorMessage = ref('')

const normalizeRequests = (payload) => {
  const rows = Array.isArray(payload) ? payload : Array.isArray(payload?.data) ? payload.data : []

  return rows.map((item) => ({
    timestamp: item.timestamp ?? item['ประทับเวลา'] ?? '-',
    fullname: item.fullname ?? item['ชื่อ-นามสกุล'] ?? '-',
    department: item.department ?? item['แผนก'] ?? '-',
    itemName: item.itemName ?? item['รายชื่อ - อุปกรณ์'] ?? item['รายชื่ออุปกรณ์'] ?? '-',
    quantity: item.quantity ?? item['จำนวนที่เบิก'] ?? '-'
  }))
}

const fetchData = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    const response = await fetch('https://chattida.app.n8n.cloud/webhook/data')

    if (!response.ok) {
      throw new Error('Failed to load request data')
    }

    const result = await response.json()
    requests.value = normalizeRequests(result)
  } catch (error) {
    console.error(error)
    errorMessage.value = 'ไม่สามารถดึงข้อมูลได้ กรุณาตรวจสอบ n8n flow หรือ webhook endpoint'
    requests.value = []
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchData()
})
</script>

<style scoped>
.request-page {
  max-width: 1240px;
  margin: 0 auto;
  display: grid;
  gap: 24px;
}

.request-hero,
.dashboard-shell {
  border-radius: 32px;
  border: 1px solid rgba(94, 67, 42, 0.12);
  box-shadow: 0 28px 70px rgba(83, 57, 33, 0.1);
  overflow: hidden;
}

.request-hero {
  padding: 36px;
  background:
    radial-gradient(circle at top right, rgba(218, 195, 162, 0.2), transparent 26%),
    linear-gradient(135deg, rgba(28, 57, 51, 0.98), rgba(64, 49, 37, 0.92));
  color: #fff8f0;
  display: grid;
  grid-template-columns: 1.5fr 0.9fr;
  gap: 22px;
}

.hero-kicker {
  margin: 0 0 14px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  font-size: 0.72rem;
  color: rgba(255, 247, 236, 0.72);
}

.request-hero h2 {
  margin: 0 0 14px;
  max-width: 720px;
  font-size: clamp(2rem, 3.8vw, 3.5rem);
  line-height: 1.08;
}

.hero-text {
  max-width: 700px;
  margin: 0;
  line-height: 1.8;
  color: rgba(255, 247, 236, 0.82);
}

.hero-summary {
  display: grid;
  gap: 14px;
}

.summary-card {
  padding: 22px;
  border-radius: 24px;
  background: rgba(255, 249, 241, 0.1);
  border: 1px solid rgba(255, 242, 230, 0.12);
}

.summary-card span {
  display: block;
  margin-bottom: 10px;
  color: rgba(255, 248, 240, 0.72);
}

.summary-card strong {
  font-size: 1.8rem;
}

.dashboard-shell {
  padding: 30px;
  background: rgba(255, 251, 246, 0.84);
  backdrop-filter: blur(14px);
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: end;
  gap: 18px;
  margin-bottom: 20px;
}

.toolbar-kicker {
  color: #9c7b54;
}

.toolbar h3 {
  margin: 0;
  font-size: 1.85rem;
}

.toolbar-text {
  margin: 8px 0 0;
  color: #75685d;
}

.refresh-btn {
  border: none;
  border-radius: 999px;
  padding: 14px 22px;
  background: linear-gradient(135deg, #204d43, #b68a52);
  color: #fff9f3;
  font-weight: 700;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  box-shadow: 0 18px 34px rgba(80, 57, 34, 0.16);
}

.refresh-btn:disabled {
  opacity: 0.7;
}

.notice {
  padding: 16px 18px;
  border-radius: 20px;
  margin-bottom: 18px;
}

.notice-error {
  background: rgba(138, 43, 43, 0.1);
  color: #8a2b2b;
}

.state-panel {
  padding: 60px 20px;
  text-align: center;
  color: #6c5f54;
}

.state-panel p,
.state-panel h4 {
  margin-bottom: 0;
}

.state-panel p {
  margin-top: 14px;
}

.table-wrap {
  overflow: auto;
  border-radius: 24px;
  border: 1px solid rgba(121, 95, 72, 0.14);
}

.request-table {
  width: 100%;
  min-width: 760px;
  border-collapse: collapse;
  background: rgba(255, 253, 249, 0.95);
}

.request-table thead th {
  padding: 18px 16px;
  background: linear-gradient(135deg, #233c36, #af8754);
  color: #fffaf3;
  font-size: 0.92rem;
  font-weight: 700;
  white-space: nowrap;
}

.request-table tbody td {
  padding: 16px;
  border-bottom: 1px solid rgba(127, 104, 83, 0.12);
  color: #342a22;
}

.request-table tbody tr:nth-child(even) td {
  background: rgba(250, 244, 236, 0.6);
}

.request-table tbody tr:hover td {
  background: rgba(221, 203, 174, 0.18);
}

.name-cell {
  font-weight: 700;
}

.badge {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  padding: 8px 14px;
  border-radius: 999px;
  font-weight: 700;
}

.badge-department {
  background: rgba(182, 138, 82, 0.14);
  color: #8c6738;
}

.badge-quantity {
  background: rgba(31, 77, 67, 0.12);
  color: #1f4d43;
}

.empty-panel {
  padding: 72px 20px;
}

.empty-mark {
  width: 72px;
  height: 72px;
  margin: 0 auto 18px;
  border-radius: 24px;
  display: grid;
  place-items: center;
  font-size: 1.3rem;
  font-weight: 700;
  color: #204d43;
  background: linear-gradient(135deg, rgba(182, 138, 82, 0.18), rgba(31, 77, 67, 0.08));
}

@media (max-width: 900px) {
  .request-hero {
    grid-template-columns: 1fr;
  }

  .toolbar {
    flex-direction: column;
    align-items: stretch;
  }

  .refresh-btn {
    width: 100%;
  }
}

@media (max-width: 640px) {
  .request-hero,
  .dashboard-shell {
    padding: 22px;
    border-radius: 24px;
  }

  .request-hero h2 {
    font-size: 2rem;
  }
}
</style>
