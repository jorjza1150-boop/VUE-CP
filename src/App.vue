<script setup>
import { ref, computed, onMounted } from 'vue'
const tabs = [{ id: 'grid', label: 'สินค้าแบบคอลัมน์' }, { id: 'products', label: 'สินค้าแบบตาราง' }, { id: 'users', label: 'ผู้ใช้แบบตาราง' }]
const active = ref('grid')
const query = ref('')
const products = ref([])
const users = ref([])
const loading = ref(false)
const errors = ref({})
const isUsers = computed(() => active.value === 'users')
const filteredProducts = computed(() => products.value.filter(p => `${p.title} ${p.category}`.toLowerCase().includes(query.value.toLowerCase().trim())))
const filteredUsers = computed(() => users.value.filter(u => `${u.firstName} ${u.lastName} ${u.username} ${u.email}`.toLowerCase().includes(query.value.toLowerCase().trim())))
const count = computed(() => isUsers.value ? filteredUsers.value.length : filteredProducts.value.length)
const error = computed(() => errors.value[isUsers.value ? 'users' : 'products'])
const money = value => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value)
function selectTab(id) { active.value = id; query.value = '' }
async function loadData() {
  if (loading.value) return
  loading.value = true
  errors.value = {}
  await Promise.all(['products', 'users'].map(async resource => {
    try {
      const response = await fetch(`https://dummyjson.com/${resource}?limit=30`, { signal: AbortSignal.timeout(15000) })
      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      const data = await response.json()
      if (!Array.isArray(data[resource])) throw new Error('ข้อมูลไม่ถูกต้อง')
      if (resource === 'products') products.value = data.products
      else users.value = data.users
    } catch {
      errors.value[resource] = 'โหลดข้อมูลไม่สำเร็จ กรุณาตรวจสอบอินเทอร์เน็ตแล้วลองอีกครั้ง'
    }
  }))
  loading.value = false
}
onMounted(loadData)
</script>

<template>
  <main>
    <header>
      <h1>สินค้าและผู้ใช้</h1>
      <p class="author">Chatree Puangpachung · 69702075</p>
      <p class="assignment">แบบฝึกหัดที่ 17 · Vue.js และ API</p>
    </header>
    <nav aria-label="รูปแบบการแสดงข้อมูล">
      <button v-for="tab in tabs" :key="tab.id" :class="{ selected: active === tab.id }" :aria-pressed="active === tab.id"
        @click="selectTab(tab.id)">{{ tab.label }}</button>
    </nav>
    <div class="toolbar">
      <label class="search"><svg viewBox="0 0 24 24" aria-hidden="true">
          <circle cx="10" cy="10" r="6.5" />
          <path d="m15 15 6 6" />
        </svg><input v-model="query" :aria-label="isUsers ? 'ค้นหาผู้ใช้' : 'ค้นหาสินค้า'"
          :placeholder="isUsers ? 'ค้นหาชื่อผู้ใช้ หรืออีเมล…' : 'ค้นหาชื่อสินค้า หรือหมวดหมู่…'"></label>
      <button class="refresh" :disabled="loading" @click="loadData"><svg viewBox="0 0 24 24" aria-hidden="true">
          <path d="M20 7a9 9 0 1 0 1 9M20 2v6h-6" />
        </svg>{{ loading ? 'กำลังโหลด…' : 'รีเฟรชข้อมูล' }}</button>
    </div>
    <div v-if="error" class="message error" role="alert">{{ error }} <button @click="loadData"
        :disabled="loading">ลองอีกครั้ง</button></div>
    <p v-if="loading" class="message" role="status">กำลังดึงข้อมูลสินค้าและผู้ใช้จาก API…</p>
    <template v-else>
      <div class="section-heading">
        <h2>{{tabs.find(t => t.id === active).label}}</h2><span>{{ count }} รายการ</span>
      </div>
      <p v-if="count === 0 && !error" class="message">ไม่พบข้อมูลที่ตรงกับการค้นหา</p>
      <div v-if="active === 'grid'" class="grid">
        <article v-for="p in filteredProducts" :key="p.id" class="product">
          <div class="photo"><img :src="p.thumbnail" :alt="p.title" loading="lazy"></div>
          <h3>{{ p.title }}</h3>
          <p class="category">{{ p.category }}</p>
          <div class="price-row"><strong>{{ money(p.price) }}</strong><span>สต็อก {{ p.stock }} ชิ้น</span></div>
        </article>
      </div>
      <div v-else class="table-wrap" tabindex="0"
        :aria-label="isUsers ? 'ตารางข้อมูลผู้ใช้ เลื่อนแนวนอนได้' : 'ตารางสินค้า เลื่อนแนวนอนได้'">
        <table v-if="!isUsers">
          <caption class="sr-only">ข้อมูลสินค้าจาก API</caption>
          <thead>
            <tr>
              <th scope="col">รหัส</th>
              <th scope="col">รูปภาพ</th>
              <th scope="col">ชื่อสินค้า</th>
              <th scope="col">หมวดหมู่</th>
              <th scope="col">ราคา (USD)</th>
              <th scope="col">สต็อก</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="p in filteredProducts" :key="p.id">
              <td>{{ p.id }}</td>
              <td><img class="thumbnail" :src="p.thumbnail" :alt="p.title" loading="lazy"></td>
              <td class="name">{{ p.title }}</td>
              <td>{{ p.category }}</td>
              <td class="price">{{ money(p.price) }}</td>
              <td>{{ p.stock }}</td>
            </tr>
          </tbody>
        </table>
        <table v-else>
          <caption class="sr-only">ข้อมูลผู้ใช้จาก API</caption>
          <thead>
            <tr>
              <th scope="col">รหัส</th>
              <th scope="col">ชื่อ–นามสกุล</th>
              <th scope="col">ชื่อผู้ใช้</th>
              <th scope="col">อีเมล</th>
              <th scope="col">โทรศัพท์</th>
              <th scope="col">เมือง</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="u in filteredUsers" :key="u.id">
              <td>{{ u.id }}</td>
              <td class="name">{{ u.firstName }} {{ u.lastName }}</td>
              <td>{{ u.username }}</td>
              <td>{{ u.email }}</td>
              <td>{{ u.phone }}</td>
              <td>{{ u.address?.city || '—' }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>
    <footer>ข้อมูลตัวอย่างจาก <a href="https://dummyjson.com" target="_blank" rel="noopener noreferrer">DummyJSON</a> ·
      ราคาเป็นดอลลาร์สหรัฐ (USD)</footer>
  </main>
</template>
