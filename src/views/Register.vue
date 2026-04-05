<template>
  <section class="register-page">
    <article class="register-intro">
      <p class="eyebrow">Smart Equipment Request</p>
      <h2>ฟอร์มเบิกอุปกรณ์ที่ดูดีขึ้น และใช้งานไวขึ้น</h2>
      <p>
        ปรับเลย์เอาต์ใหม่ให้ส่วนกรอกข้อมูลอ่านง่ายขึ้น แยก hierarchy ชัด และใช้โทนเดียวกับทั้งระบบ
        เพื่อให้ประสบการณ์ใช้งานดูเป็นมืออาชีพมากขึ้น
      </p>

      <div class="intro-grid">
        <div class="intro-box">
          <span>Flow</span>
          <strong>n8n + Google Sheets</strong>
        </div>
        <div class="intro-box">
          <span>Status</span>
          <strong>{{ loading ? 'กำลังส่งคำขอ' : 'พร้อมรับข้อมูล' }}</strong>
        </div>
      </div>
    </article>

    <section class="register-card">
      <div class="register-card__header">
        <div>
          <p class="eyebrow eyebrow-dark">Request Form</p>
          <h3>แบบฟอร์มเบิกอุปกรณ์</h3>
        </div>
        <p>ส่งข้อมูลไปยัง n8n เพื่อบันทึกลง Google Sheets อัตโนมัติ</p>
      </div>

      <transition name="rise">
        <div v-if="status.message" :class="['feedback', `feedback-${status.type}`]">
          {{ status.message }}
        </div>
      </transition>

      <form class="register-form" @submit.prevent="submitForm">
        <label class="form-item">
          <span>ชื่อ - นามสกุล</span>
          <input v-model="form.fullname" type="text" placeholder="กรอกชื่อผู้เบิก" required />
        </label>

        <label class="form-item">
          <span>แผนก</span>
          <select v-model="form.department" required>
            <option value="" disabled>เลือกแผนก</option>
            <option value="บุคคล">บุคคล</option>
            <option value="บัญชี">บัญชี</option>
            <option value="ไอที">ไอที</option>
            <option value="การตลาด">การตลาด</option>
            <option value="ประชาสัมพันธ์">ประชาสัมพันธ์</option>
          </select>
        </label>

        <label class="form-item form-item-full">
          <span>รายการอุปกรณ์</span>
          <select v-model="form.itemName" required>
            <option value="" disabled>เลือกอุปกรณ์</option>
            <option value="กรรไกร">กรรไกร</option>
            <option value="เทปกาว">เทปกาว</option>
            <option value="ปากกา">ปากกา</option>
            <option value="แฟ้มเอกสาร">แฟ้มเอกสาร</option>
            <option value="คัตเตอร์">คัตเตอร์</option>
          </select>
        </label>

        <label class="form-item form-item-qty">
          <span>จำนวนที่เบิก</span>
          <input v-model.number="form.quantity" type="number" min="1" placeholder="1" required />
        </label>

        <div class="register-actions">
          <p>ระบบจะบันทึกเวลาเป็นรูปแบบภาษาไทยให้อัตโนมัติ</p>
          <button class="request-btn" :disabled="loading">
            <span v-if="loading" class="spinner-border spinner-border-sm"></span>
            <span>{{ loading ? 'กำลังบันทึกข้อมูล...' : 'บันทึกการเบิกอุปกรณ์' }}</span>
          </button>
        </div>
      </form>
    </section>
  </section>
</template>

<script setup>
import { reactive, ref } from 'vue'

const form = reactive({
  fullname: '',
  department: '',
  itemName: '',
  quantity: null
})

const loading = ref(false)

const status = reactive({
  message: '',
  type: ''
})

const resetForm = () => {
  form.fullname = ''
  form.department = ''
  form.itemName = ''
  form.quantity = null
}

const submitForm = async () => {
  loading.value = true
  status.message = ''

  try {
    const response = await fetch('http://localhost:5678/webhook/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        ...form,
        timestamp: new Date().toLocaleString('th-TH')
      })
    })

    if (!response.ok) {
      throw new Error('Request failed')
    }

    await response.json()
    status.message = 'บันทึกข้อมูลการเบิกอุปกรณ์สำเร็จ'
    status.type = 'success'
    resetForm()
  } catch (error) {
    console.error(error)
    status.message = 'เกิดข้อผิดพลาด กรุณาตรวจสอบ n8n flow แล้วลองใหม่อีกครั้ง'
    status.type = 'danger'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.register-page {
  max-width: 1240px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1fr 1.2fr;
  gap: 24px;
}

.register-intro,
.register-card {
  border-radius: 32px;
  border: 1px solid rgba(94, 67, 42, 0.12);
  box-shadow: 0 28px 70px rgba(83, 57, 33, 0.1);
  overflow: hidden;
}

.register-intro {
  padding: 36px;
  background:
    radial-gradient(circle at top right, rgba(182, 138, 82, 0.2), transparent 28%),
    linear-gradient(145deg, rgba(31, 77, 67, 0.98), rgba(43, 34, 28, 0.94));
  color: #fff8f0;
}

.eyebrow {
  margin: 0 0 14px;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  font-size: 0.72rem;
  color: rgba(255, 247, 236, 0.72);
}

.eyebrow-dark {
  color: #97764f;
}

.register-intro h2 {
  margin: 0 0 16px;
  font-size: clamp(2rem, 4vw, 3.4rem);
  line-height: 1.1;
}

.register-intro p {
  margin: 0;
  line-height: 1.8;
  color: rgba(255, 248, 240, 0.84);
}

.intro-grid {
  display: grid;
  gap: 14px;
  margin-top: 28px;
}

.intro-box {
  padding: 18px 20px;
  border-radius: 22px;
  background: rgba(255, 249, 241, 0.1);
  border: 1px solid rgba(255, 241, 223, 0.12);
}

.intro-box span {
  display: block;
  margin-bottom: 8px;
  color: rgba(255, 248, 240, 0.72);
}

.intro-box strong {
  font-size: 1.15rem;
}

.register-card {
  padding: 32px;
  background: rgba(255, 251, 246, 0.84);
  backdrop-filter: blur(14px);
}

.register-card__header {
  display: flex;
  justify-content: space-between;
  align-items: end;
  gap: 18px;
  margin-bottom: 24px;
}

.register-card__header h3 {
  margin: 0;
  font-size: 1.9rem;
}

.register-card__header p {
  max-width: 320px;
  margin: 0;
  color: #75685d;
  line-height: 1.7;
  text-align: right;
}

.feedback {
  margin-bottom: 20px;
  padding: 16px 18px;
  border-radius: 20px;
  font-weight: 600;
}

.feedback-success {
  background: rgba(31, 77, 67, 0.1);
  color: #1f4d43;
}

.feedback-danger {
  background: rgba(138, 43, 43, 0.1);
  color: #8a2b2b;
}

.register-form {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
}

.form-item {
  display: grid;
  gap: 10px;
}

.form-item-full,
.register-actions {
  grid-column: 1 / -1;
}

.form-item-qty {
  max-width: 220px;
}

.form-item span {
  font-weight: 600;
  color: #49382c;
}

.form-item input,
.form-item select {
  width: 100%;
  padding: 16px 18px;
  border: 1px solid rgba(116, 90, 68, 0.16);
  border-radius: 18px;
  background: rgba(255, 254, 251, 0.92);
  color: #231815;
  transition: border-color 0.2s ease, box-shadow 0.2s ease, transform 0.2s ease;
}

.form-item input::placeholder {
  color: #9b8a7a;
}

.form-item input:focus,
.form-item select:focus {
  outline: none;
  border-color: #b68a52;
  box-shadow: 0 0 0 4px rgba(182, 138, 82, 0.14);
  transform: translateY(-1px);
}

.register-actions {
  margin-top: 6px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 18px;
}

.register-actions p {
  margin: 0;
  color: #75685d;
}

.request-btn {
  min-width: 220px;
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

.request-btn:disabled {
  opacity: 0.72;
}

.rise-enter-active,
.rise-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}

.rise-enter-from,
.rise-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}

@media (max-width: 980px) {
  .register-page {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 720px) {
  .register-card__header,
  .register-actions {
    flex-direction: column;
    align-items: flex-start;
  }

  .register-card__header p {
    max-width: none;
    text-align: left;
  }
}

@media (max-width: 640px) {
  .register-intro,
  .register-card {
    padding: 22px;
    border-radius: 24px;
  }

  .register-form {
    grid-template-columns: 1fr;
  }

  .form-item-full,
  .register-actions {
    grid-column: auto;
  }

  .form-item-qty {
    max-width: none;
  }

  .request-btn {
    width: 100%;
  }
}
</style>
