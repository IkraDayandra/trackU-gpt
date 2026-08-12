<script setup lang="ts">
import { computed, ref } from 'vue'

type Nav = 'Dashboard' | 'Transaksi' | 'Dompet' | 'Kategori' | 'Anggaran' | 'Target tabungan' | 'Asisten AI' | 'Pengaturan'
type Transaction = { id: number; title: string; category: string; wallet: string; amount: number; type: 'income' | 'expense'; date: string; icon: string; color: string }

const navItems: { label: Nav; icon: string }[] = [
  { label: 'Dashboard', icon: 'grid' }, { label: 'Transaksi', icon: 'receipt' }, { label: 'Dompet', icon: 'wallet' },
  { label: 'Kategori', icon: 'tag' }, { label: 'Anggaran', icon: 'pie' }, { label: 'Target tabungan', icon: 'target' },
  { label: 'Asisten AI', icon: 'sparkle' }, { label: 'Pengaturan', icon: 'settings' },
]

const active = ref<Nav>('Dashboard')
const mobileMenu = ref(false)
const modalOpen = ref(false)
const showAllTransactions = ref(false)
const search = ref('')
const transactionType = ref<'all' | 'income' | 'expense'>('all')
const toast = ref('')
const question = ref('')
const assistantMessages = ref([{ from: 'ai', text: 'Hai Ikra! Aku bisa membantumu memahami keuangan grup. Mau tahu apa hari ini?' }])
const selectedGroup = ref('Keuangan Rumah')

const transactions = ref<Transaction[]>([
  { id: 1, title: 'Gaji bulanan', category: 'Gaji', wallet: 'BCA Utama', amount: 12500000, type: 'income', date: 'Hari ini, 08.30', icon: 'briefcase', color: 'purple' },
  { id: 2, title: 'Belanja mingguan', category: 'Belanja kebutuhan', wallet: 'BCA Utama', amount: 784500, type: 'expense', date: 'Hari ini, 07.20', icon: 'bag', color: 'orange' },
  { id: 3, title: 'Tagihan internet', category: 'Tagihan & utilitas', wallet: 'Mandiri Bersama', amount: 449000, type: 'expense', date: 'Kemarin', icon: 'wifi', color: 'blue' },
  { id: 4, title: 'Proyek freelance', category: 'Freelance', wallet: 'Tabungan Jago', amount: 2850000, type: 'income', date: '12 Agu', icon: 'star', color: 'yellow' },
  { id: 5, title: 'Makan malam keluarga', category: 'Makan & minum', wallet: 'Tunai', amount: 326000, type: 'expense', date: '11 Agu', icon: 'utensils', color: 'red' },
  { id: 6, title: 'Isi bensin', category: 'Transportasi', wallet: 'BCA Utama', amount: 200000, type: 'expense', date: '10 Agu', icon: 'car', color: 'teal' },
])

const form = ref({ title: '', amount: '', type: 'expense' as 'income' | 'expense', category: 'Makan & minum', wallet: 'BCA Utama' })

const filteredTransactions = computed(() => transactions.value.filter((item) => {
  const query = search.value.toLowerCase()
  return (transactionType.value === 'all' || item.type === transactionType.value) && (!query || `${item.title} ${item.category} ${item.wallet}`.toLowerCase().includes(query))
}))
const visibleTransactions = computed(() => showAllTransactions.value ? filteredTransactions.value : filteredTransactions.value.slice(0, 5))
const currentMonthExpense = computed(() => transactions.value.filter(t => t.type === 'expense').reduce((sum, t) => sum + t.amount, 0))
const balance = computed(() => transactions.value.reduce((sum, t) => sum + (t.type === 'income' ? t.amount : -t.amount), 18436200))

function format(value: number, compact = false) {
  if (compact && value >= 1000000) return `Rp${(value / 1000000).toFixed(value % 1000000 === 0 ? 0 : 1)}M`
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', maximumFractionDigits: 0 }).format(value)
}
function selectPage(page: Nav) { active.value = page; mobileMenu.value = false; window.scrollTo({ top: 0, behavior: 'smooth' }) }
function typeLabel(type: 'all' | 'income' | 'expense') { return type === 'all' ? 'Semua' : type === 'income' ? 'Pemasukan' : 'Pengeluaran' }
function addTransaction() {
  const amount = Number(form.value.amount.replace(/[^0-9]/g, ''))
  if (!form.value.title || !amount) { notify('Isi judul dan nominal terlebih dahulu'); return }
  transactions.value.unshift({ id: Date.now(), title: form.value.title, amount, type: form.value.type, category: form.value.category, wallet: form.value.wallet, date: 'Baru saja', icon: form.value.type === 'income' ? 'plus' : 'receipt', color: form.value.type === 'income' ? 'purple' : 'orange' })
  modalOpen.value = false; form.value = { title: '', amount: '', type: 'expense', category: 'Makan & minum', wallet: 'BCA Utama' }; notify('Transaksi ditambahkan ke Keuangan Rumah')
}
function notify(message: string) { toast.value = message; window.setTimeout(() => toast.value = '', 2800) }
function askAssistant(suggestion?: string) {
  const text = suggestion || question.value
  if (!text.trim()) return
  assistantMessages.value.push({ from: 'user', text })
  question.value = ''
  window.setTimeout(() => assistantMessages.value.push({ from: 'ai', text: 'Bulan ini, Makan & minum adalah pengeluaran terbesar kamu sebesar Rp1,24 jt. Sisa anggarannya masih Rp760 rb—kamu sudah cukup baik menjaga pengeluaran.' }), 350)
}
</script>

<template>
  <div class="app-shell">
    <aside class="sidebar" :class="{ open: mobileMenu }">
      <div class="brand"><span class="brand-mark">↑</span><span>trackU</span></div>
      <button class="group-switcher" @click="selectedGroup = selectedGroup === 'Keuangan Rumah' ? 'Keuangan Pribadi' : 'Keuangan Rumah'">
        <span class="group-avatar">KR</span><span class="group-copy"><b>{{ selectedGroup }}</b><small>Pemilik</small></span><span class="chevron">⌄</span>
      </button>
      <nav>
        <button v-for="item in navItems" :key="item.label" class="nav-item" :class="{ active: active === item.label }" @click="selectPage(item.label)"><span class="nav-icon">{{ item.icon === 'grid' ? '⊞' : item.icon === 'receipt' ? '▤' : item.icon === 'wallet' ? '▱' : item.icon === 'tag' ? '◇' : item.icon === 'pie' ? '◔' : item.icon === 'target' ? '◎' : item.icon === 'sparkle' ? '✦' : '⚙' }}</span>{{ item.label }}</button>
      </nav>
      <div class="sidebar-bottom"><button class="invite" @click="notify('Tautan undangan berhasil disalin')"><span>＋</span> Undang anggota</button><div class="profile"><div class="avatar">ID</div><div><b>Ikra Dayandra</b><small>ikra@email.com</small></div><button @click="notify('Sesi demo telah keluar')">↪</button></div></div>
    </aside>
    <div v-if="mobileMenu" class="backdrop" @click="mobileMenu = false"></div>

    <main>
      <header class="topbar">
        <button class="mobile-toggle" @click="mobileMenu = !mobileMenu">☰</button>
        <div class="mobile-brand">trackU</div>
        <div class="period"><span class="calendar">□</span><span>Agustus 2026</span><span class="chevron">⌄</span></div>
        <div class="top-actions"><button class="icon-button" @click="notify('Tidak ada notifikasi baru')">♧<i></i></button><button class="help" @click="notify('Pusat bantuan segera hadir')">?</button></div>
      </header>

      <section v-if="active === 'Dashboard'" class="page dashboard">
        <div class="page-heading"><div><p class="eyebrow">SELASA, 12 AGUSTUS</p><h1>Selamat pagi, Ikra <span>✦</span></h1><p class="subcopy">Ini ringkasan kondisi keuanganmu bulan ini.</p></div><button class="primary-button" @click="modalOpen = true"><span>＋</span> Tambah transaksi</button></div>
        <div class="summary-grid">
          <article class="summary-card balance"><div class="card-label"><span>Total saldo</span><span class="dots">•••</span></div><strong>{{ format(balance) }}</strong><div class="summary-footer"><span class="trend good">↗ 12,5%</span><span>dari bulan lalu</span></div><div class="orb orb-one"></div><div class="orb orb-two"></div></article>
          <article class="summary-card"><div class="card-label"><span>Pemasukan</span><span class="metric-icon income">↙</span></div><strong>{{ format(15350000) }}</strong><div class="summary-footer"><span class="trend good">↗ 8,2%</span><span>dari bulan lalu</span></div></article>
          <article class="summary-card"><div class="card-label"><span>Pengeluaran</span><span class="metric-icon expense">↗</span></div><strong>{{ format(currentMonthExpense) }}</strong><div class="summary-footer"><span class="trend bad">↗ 4,1%</span><span>dari bulan lalu</span></div></article>
          <article class="summary-card"><div class="card-label"><span>Arus kas</span><span class="metric-icon flow">≋</span></div><strong>{{ format(11240500) }}</strong><div class="summary-footer"><span class="trend good">↗ 15,3%</span><span>dari bulan lalu</span></div></article>
        </div>
        <div class="dashboard-grid">
          <section class="panel cashflow-panel"><div class="panel-heading"><div><h2>Arus kas</h2><p>Pemasukan dan pengeluaran dari waktu ke waktu</p></div><button class="select-button">Bulan ini <span>⌄</span></button></div><div class="chart-legend"><span><i class="legend income-dot"></i> Pemasukan</span><span><i class="legend expense-dot"></i> Pengeluaran</span></div><div class="line-chart"><div class="chart-lines"><i v-for="n in 5" :key="n"></i></div><div class="area-fill"></div><svg viewBox="0 0 620 170" preserveAspectRatio="none" aria-label="Grafik pemasukan dan pengeluaran"><polyline class="income-line" points="0,130 65,120 122,128 185,90 245,101 310,72 372,83 435,48 495,58 555,24 620,34"/><polyline class="expense-line" points="0,144 65,137 122,142 185,123 245,135 310,112 372,119 435,103 495,110 555,89 620,96"/></svg><div class="chart-labels"><span>1 Agu</span><span>5 Agu</span><span>10 Agu</span><span>15 Agu</span><span>20 Agu</span><span>25 Agu</span><span>31 Agu</span></div></div></section>
          <section class="panel spending-panel"><div class="panel-heading"><div><h2>Pengeluaran per kategori</h2><p>Agustus 2026</p></div><button class="dots">•••</button></div><div class="donut-wrap"><div class="donut"><div><strong>Rp4,1 jt</strong><span>Total pengeluaran</span></div></div><div class="category-list"><div><span><i class="color-dot food"></i>Makan & minum</span><b>Rp1,24 jt</b></div><div><span><i class="color-dot transport"></i>Transportasi</span><b>Rp760 rb</b></div><div><span><i class="color-dot bills"></i>Tagihan & utilitas</span><b>Rp680 rb</b></div><div><span><i class="color-dot shop"></i>Belanja</span><b>Rp520 rb</b></div><div><span><i class="color-dot more"></i>Lainnya</span><b>Rp916 rb</b></div></div></div></section>
        </div>
        <div class="lower-grid"><section class="panel transactions-panel"><div class="panel-heading"><div><h2>Transaksi terbaru</h2><p>Aktivitas terbarumu</p></div><button class="text-button" @click="active = 'Transaksi'; showAllTransactions = true">Lihat semua <span>→</span></button></div><TransactionList :items="transactions.slice(0, 5)" :format="format" /></section><section class="panel progress-panel"><div class="panel-heading"><div><h2>Sesuai rencana</h2><p>Rencana keuangan bulan ini</p></div><button class="text-button" @click="active = 'Anggaran'">Lihat semua <span>→</span></button></div><div class="progress-item"><div class="progress-top"><span class="round-icon food-bg">♨</span><div><b>Makan & minum</b><small>Rp1,24 jt dari Rp2 jt</small></div><em>62%</em></div><div class="progress-bar"><i style="width:62%"></i></div></div><div class="progress-item"><div class="progress-top"><span class="round-icon goal-bg">◎</span><div><b>Dana darurat</b><small>Rp8,4 jt dari Rp15 jt</small></div><em>56%</em></div><div class="progress-bar goal"><i style="width:56%"></i></div></div></section></div>
      </section>

      <section v-else-if="active === 'Transaksi'" class="page transaction-page"><div class="page-heading"><div><p class="eyebrow">AGUSTUS 2026</p><h1>Transaksi</h1><p class="subcopy">Catat setiap pergerakan uang di satu tempat.</p></div><button class="primary-button" @click="modalOpen = true"><span>＋</span> Tambah transaksi</button></div><div class="filter-bar"><label class="search"><span>⌕</span><input v-model="search" placeholder="Cari transaksi" /></label><div class="type-filter"><button v-for="option in ['all', 'income', 'expense']" :key="option" :class="{ selected: transactionType === option }" @click="transactionType = option as 'all' | 'income' | 'expense'">{{ typeLabel(option as 'all' | 'income' | 'expense') }}</button></div></div><section class="panel table-panel"><div class="table-title"><h2>{{ filteredTransactions.length }} transaksi</h2><button class="select-button">Semua dompet ⌄</button></div><TransactionList :items="visibleTransactions" :format="format" full /><button v-if="filteredTransactions.length > 5 && !showAllTransactions" class="show-more" @click="showAllTransactions = true">Tampilkan semua transaksi</button></section></section>

      <section v-else-if="active === 'Asisten AI'" class="page assistant-page"><div class="page-heading"><div><p class="eyebrow">KECERDASAN TRACKU</p><h1>Asisten keuangan AI <span>✦</span></h1><p class="subcopy">Ajukan pertanyaan tentang data keuangan bersama dengan bahasa sehari-hari.</p></div></div><div class="assistant-layout"><section class="assistant-chat panel"><div class="assistant-head"><span class="assistant-icon">✦</span><div><b>Asisten trackU</b><small>Menggunakan data dari {{ selectedGroup }}</small></div><span class="online">● Aktif</span></div><div class="messages"><div v-for="(message, index) in assistantMessages" :key="index" class="message" :class="message.from"><span v-if="message.from === 'ai'" class="bot-dot">✦</span><p>{{ message.text }}</p></div></div><div class="suggestions"><button @click="askAssistant('Pengeluaran terbesar saya apa?')">Pengeluaran terbesar saya apa?</button><button @click="askAssistant('Bagaimana kondisi anggaran saya?')">Bagaimana kondisi anggaran saya?</button></div><form class="ask-box" @submit.prevent="askAssistant()"><input v-model="question" placeholder="Tanyakan tentang keuanganmu…" /><button aria-label="Kirim pertanyaan">↑</button></form></section><aside class="panel assistant-side"><span class="big-sparkle">✦</span><h2>Dibuat dari datamu.</h2><p>Asisten dapat merangkum pengeluaran, memeriksa anggaran, dan menemukan pola di grup ini.</p><div class="privacy-note"><span>⌾</span><p>Hanya data keuangan yang boleh kamu akses di grup ini yang digunakan untuk menjawab pertanyaanmu.</p></div></aside></div></section>

      <section v-else class="page placeholder-page"><div class="page-heading"><div><p class="eyebrow">{{ active.toUpperCase() }}</p><h1>{{ active }}</h1><p class="subcopy">Kelola {{ active.toLowerCase() }} untuk {{ selectedGroup }}.</p></div><button v-if="active !== 'Pengaturan'" class="primary-button" @click="notify(`Editor ${active.toLowerCase()} siap dihubungkan ke backend`) ">＋ Tambah {{ active === 'Target tabungan' ? 'target' : active.slice(0, -1).toLowerCase() }}</button></div><section class="empty-feature panel"><div class="feature-mark">{{ active === 'Dompet' ? '▱' : active === 'Anggaran' ? '◔' : active === 'Target tabungan' ? '◎' : active === 'Kategori' ? '◇' : '⚙' }}</div><h2>Ringkasan {{ active.toLowerCase() }}</h2><p>Tampilan frontend ini sudah siap dihubungkan ke API khusus nantinya. Saat ini, dashboard dan transaksi memakai data contoh yang dapat berinteraksi.</p><div class="preview-cards"><div><small>Grup aktif</small><b>{{ selectedGroup }}</b></div><div><small>Status</small><b class="ready">Siap dihubungkan</b></div></div></section></section>
    </main>

    <div v-if="modalOpen" class="modal-layer" @click.self="modalOpen = false"><form class="modal" @submit.prevent="addTransaction"><button type="button" class="modal-close" @click="modalOpen = false">×</button><p class="eyebrow">CATATAN BARU</p><h2>Tambah transaksi</h2><p class="modal-copy">Data ini hanya disimpan di demo lokal.</p><div class="form-type"><button v-for="kind in ['expense', 'income']" :key="kind" type="button" :class="{ chosen: form.type === kind }" @click="form.type = kind as 'income' | 'expense'">{{ typeLabel(kind as 'income' | 'expense') }}</button></div><label>Judul<input v-model="form.title" placeholder="Contoh: Makan siang di Hara" autofocus /></label><label>Nominal<input v-model="form.amount" inputmode="numeric" placeholder="0" /></label><div class="form-row"><label>Kategori<select v-model="form.category"><option>Makan & minum</option><option>Transportasi</option><option>Belanja kebutuhan</option><option>Gaji</option><option>Freelance</option></select></label><label>Dompet<select v-model="form.wallet"><option>BCA Utama</option><option>Mandiri Bersama</option><option>Tabungan Jago</option><option>Tunai</option></select></label></div><button class="primary-button form-submit">Simpan transaksi <span>→</span></button></form></div>
    <transition name="toast"><div v-if="toast" class="toast">✓ {{ toast }}</div></transition>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  components: {
    TransactionList: defineComponent({
      props: { items: { type: Array, required: true }, format: { type: Function, required: true }, full: Boolean },
      template: `<div class="transaction-list"><div v-for="item in items" :key="item.id" class="transaction-row"><span class="transaction-icon" :class="item.color">{{ item.icon === 'briefcase' ? '▣' : item.icon === 'bag' ? '▰' : item.icon === 'wifi' ? '⌁' : item.icon === 'star' ? '★' : item.icon === 'utensils' ? '♨' : item.icon === 'car' ? '▱' : item.icon === 'plus' ? '＋' : '▤' }}</span><div class="transaction-name"><b>{{ item.title }}</b><span>{{ item.category }} · {{ item.wallet }}</span></div><span v-if="full" class="transaction-date">{{ item.date }}</span><div class="transaction-amount" :class="item.type"><b>{{ item.type === 'income' ? '+' : '−' }}{{ format(item.amount) }}</b><span v-if="!full">{{ item.date }}</span></div></div><p v-if="!items.length" class="no-results">Tidak ada transaksi yang sesuai dengan filter.</p></div>`
    })
  }
})
</script>
