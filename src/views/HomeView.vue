<template>
  <section class="lux-page">
    <div class="hero-grid">
      <article class="hero-card intro-card">
        <p class="section-kicker">Luxury Intake Experience</p>
        <h2 class="hero-title">ลงทะเบียนข้อมูลเข้าสู่ระบบอัตโนมัติอย่างมีระดับ</h2>
        <p class="hero-copy">
          ฟอร์มนี้ออกแบบใหม่ให้ดูเรียบหรู อ่านง่าย และโฟกัสการกรอกข้อมูลสำคัญก่อนส่งต่อไปยัง
          n8n webhook เพื่อเชื่อม workflow ได้ทันที
        </p>

        <div class="feature-row">
          <div class="feature-chip">
            <strong>Fast Flow</strong>
            <span>กรอกน้อย แต่ครบ</span>
          </div>
          <div class="feature-chip">
            <strong>Elegant UI</strong>
            <span>สไตล์ครีมทองเขียวเข้ม</span>
          </div>
        </div>
      </article>

      <article class="hero-card metric-card">
        <p class="section-kicker">Today Snapshot</p>
        <div class="metric-stack">
          <div>
            <span class="metric-label">สถานะระบบ</span>
            <strong class="metric-value">{{ loading ? 'กำลังส่งข้อมูล' : 'พร้อมใช้งาน' }}</strong>
          </div>
          <div>
            <span class="metric-label">วันที่บันทึก</span>
            <strong class="metric-value">{{ todayLabel }}</strong>
          </div>
        </div>
      </article>
    </div>

    <section class="form-shell">
      <div class="form-shell__header">
        <div>
          <p class="section-kicker">Registration Form</p>
          <h3>บันทึกข้อมูลนักศึกษา</h3>
        </div>
        <p class="header-note">ข้อมูลจะถูกส่งไปยัง n8n เพื่อใช้งานต่อใน workflow</p>
      </div>

      <transition name="fade">
        <div v-if="status.message" :class="['status-banner', `is-${status.type}`]">
          <div>
            <strong>{{ status.type === 'success' ? 'สำเร็จ' : 'เกิดข้อผิดพลาด' }}</strong>
            <p>{{ status.message }}</p>
          </div>
          <button type="button" class="dismiss-btn" @click="status.message = ''">ปิด</button>
        </div>
      </transition>

      <form class="lux-form" @submit.prevent="submitForm">
        <label class="field-block">
          <span>รหัสนักศึกษา</span>
          <input v-model="data.id" type="text" placeholder="เช่น 67705261" required />
        </label>

        <label class="field-block">
          <span>ชื่อ - นามสกุล</span>
          <input v-model="data.fullname" type="text" placeholder="กรอกชื่อผู้ลงทะเบียน" required />
        </label>

        <label class="field-block field-block--full">
          <span>สาขาวิชา / แผนก</span>
          <select v-model="data.department" required>
            <option value="" disabled>เลือกแผนก</option>
            <option value="บริหารธุรกิจ">บริหารธุรกิจ</option>
            <option value="บัญชี">บัญชี</option>
            <option value="เทคโนโลยีสารสนเทศ">เทคโนโลยีสารสนเทศ</option>
          </select>
        </label>

        <div class="form-footer">
          <p>เมื่อกดบันทึก ระบบจะเพิ่มวันที่ปัจจุบันให้อัตโนมัติ</p>
          <button class="submit-btn" :disabled="loading">
            <span v-if="loading" class="spinner-border spinner-border-sm"></span>
            <span>{{ loading ? 'กำลังส่งข้อมูล...' : 'บันทึกข้อมูล' }}</span>
          </button>
        </div>
      </form>
    </section>
  </section>
</template>

<script setup>
import { computed, reactive, ref } from 'vue'

const data = reactive({
  id: '',
  fullname: '',
  department: ''
})

const loading = ref(false)

const status = reactive({
  message: '',
  type: ''
})

const todayLabel = computed(() =>
  new Date().toLocaleDateString('th-TH', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
)

const submitForm = async () => {
  loading.value = true
  status.message = ''

  try {
    const response = await fetch('http://localhost:5678/webhook-test/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        ...data,
        tdate: new Date().toISOString().split('T')[0]
      })
    })

    if (!response.ok) {
      throw new Error('Error')
    }

    await response.json()

    status.message = 'บันทึกข้อมูลเรียบร้อยแล้ว'
    status.type = 'success'

    data.id = ''
    data.fullname = ''
    data.department = ''
  } catch (error) {
    console.error(error)
    status.message = 'ไม่สามารถส่งข้อมูลได้ กรุณาตรวจสอบ n8n webhook แล้วลองใหม่อีกครั้ง'
    status.type = 'danger'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.lux-page {
  max-width: 1240px;
  margin: 0 auto;
  display: grid;
  gap: 24px;
}

.hero-grid {
  display: grid;
  grid-template-columns: 1.6fr 0.9fr;
  gap: 24px;
}

.hero-card,
.form-shell {
  position: relative;
  overflow: hidden;
  border: 1px solid rgba(94, 67, 42, 0.12);
  border-radius: 32px;
  background: rgba(255, 251, 246, 0.82);
  box-shadow: 0 30px 70px rgba(83, 57, 33, 0.1);
  backdrop-filter: blur(14px);
}

.hero-card::before,
.form-shell::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.48), transparent 45%);
  pointer-events: none;
}

.intro-card {
  padding: 38px;
  background:
    radial-gradient(circle at top right, rgba(182, 138, 82, 0.18), transparent 30%),
    linear-gradient(140deg, rgba(29, 79, 69, 0.98), rgba(42, 35, 28, 0.94));
  color: #fff9f2;
}

.metric-card {
  padding: 32px;
}

.section-kicker {
  margin: 0 0 14px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  font-size: 0.72rem;
  color: rgba(255, 248, 239, 0.72);
}

.metric-card .section-kicker,
.form-shell .section-kicker {
  color: #8c6d4a;
}

.hero-title {
  margin: 0 0 16px;
  max-width: 700px;
  font-size: clamp(2.2rem, 4vw, 4rem);
  line-height: 1.06;
}

.hero-copy {
  max-width: 680px;
  margin: 0;
  font-size: 1rem;
  line-height: 1.8;
  color: rgba(255, 248, 239, 0.82);
}

.feature-row {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  margin-top: 30px;
}

.feature-chip {
  min-width: 180px;
  padding: 16px 18px;
  border-radius: 22px;
  background: rgba(255, 249, 241, 0.12);
  border: 1px solid rgba(255, 240, 224, 0.12);
}

.feature-chip strong,
.feature-chip span {
  display: block;
}

.feature-chip strong {
  margin-bottom: 6px;
}

.feature-chip span {
  color: rgba(255, 248, 239, 0.76);
}

.metric-stack {
  display: grid;
  gap: 24px;
  margin-top: 18px;
}

.metric-label {
  display: block;
  margin-bottom: 8px;
  color: #8b7665;
  font-size: 0.86rem;
  text-transform: uppercase;
  letter-spacing: 0.12em;
}

.metric-value {
  font-size: 1.6rem;
  line-height: 1.2;
  color: #231815;
}

.form-shell {
  padding: 34px;
}

.form-shell__header {
  display: flex;
  justify-content: space-between;
  align-items: end;
  gap: 18px;
  margin-bottom: 24px;
}

.form-shell__header h3 {
  margin: 0;
  font-size: 2rem;
}

.header-note {
  max-width: 320px;
  margin: 0;
  color: #6e6259;
  line-height: 1.7;
  text-align: right;
}

.status-banner {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  padding: 18px 20px;
  border-radius: 22px;
  margin-bottom: 22px;
}

.status-banner strong {
  display: block;
  margin-bottom: 4px;
}

.status-banner p {
  margin: 0;
}

.status-banner.is-success {
  background: rgba(31, 77, 67, 0.1);
  color: #1f4d43;
}

.status-banner.is-danger {
  background: rgba(138, 43, 43, 0.1);
  color: #8a2b2b;
}

.dismiss-btn {
  border: none;
  background: transparent;
  color: inherit;
}

.lux-form {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
}

.field-block {
  display: grid;
  gap: 10px;
}

.field-block--full {
  grid-column: 1 / -1;
}

.field-block span {
  font-size: 0.92rem;
  font-weight: 600;
  color: #49382c;
}

.field-block input,
.field-block select {
  width: 100%;
  padding: 16px 18px;
  border: 1px solid rgba(116, 90, 68, 0.16);
  border-radius: 18px;
  background: rgba(255, 254, 251, 0.92);
  color: #231815;
  transition: border-color 0.2s ease, box-shadow 0.2s ease, transform 0.2s ease;
}

.field-block input::placeholder {
  color: #9b8a7a;
}

.field-block input:focus,
.field-block select:focus {
  outline: none;
  border-color: #b68a52;
  box-shadow: 0 0 0 4px rgba(182, 138, 82, 0.14);
  transform: translateY(-1px);
}

.form-footer {
  grid-column: 1 / -1;
  margin-top: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 18px;
}

.form-footer p {
  margin: 0;
  color: #75685d;
}

.submit-btn {
  min-width: 180px;
  padding: 14px 24px;
  border: none;
  border-radius: 999px;
  background: linear-gradient(135deg, #204d43, #b68a52);
  color: #fffaf4;
  font-weight: 700;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  box-shadow: 0 18px 30px rgba(80, 57, 34, 0.16);
}

.submit-btn:disabled {
  opacity: 0.72;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}

@media (max-width: 900px) {
  .hero-grid {
    grid-template-columns: 1fr;
  }

  .form-shell__header,
  .form-footer {
    flex-direction: column;
    align-items: flex-start;
  }

  .header-note {
    max-width: none;
    text-align: left;
  }
}

@media (max-width: 640px) {
  .intro-card,
  .metric-card,
  .form-shell {
    padding: 22px;
    border-radius: 24px;
  }

  .lux-form {
    grid-template-columns: 1fr;
  }

  .field-block--full {
    grid-column: auto;
  }

  .submit-btn {
    width: 100%;
  }
}
</style>
