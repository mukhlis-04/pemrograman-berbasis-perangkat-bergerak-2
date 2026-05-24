<template>
  <ion-page>

    <ion-header class="ion-no-border">
      <ion-toolbar>
        <ion-title>☁️Cuaca Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">

      <div v-if="loading" class="state-box">
        <ion-spinner name="crescent" />
        <p>Memuat data cuaca...</p>
      </div>

      <div v-else-if="error" class="state-box">
        <p class="err-text">⚠️ {{ error }}</p>
        <!-- @click = panggil fungsi saat tombol ditekan -->
        <ion-button @click="ambilData" fill="outline" size="small">
          Coba Lagi
        </ion-button>
      </div>

      <div v-else class="konten">
        <div class="kartu-utama">
          <div class="angka-besar">{{ suhuSekarang }}°</div>
          <div class="label-kecil">Suhu saat ini · Jakarta</div>

          <div class="baris-stat">
            <div class="stat">
              <span class="stat-angka">{{ minSuhu }}°</span>
              <span class="stat-label">Terendah</span>
            </div>

            <div class="stat">
              <span class="stat-angka">{{ maxSuhu }}°</span>
              <span class="stat-label">Tertinggi</span>
            </div>

            <div class="stat">
              <span class="stat-angka">{{ rataRata }}°</span>
              <span class="stat-label">Rata-rata</span>
            </div>
          </div>
        </div>

        <ion-item class="pilih-hari">
          <ion-label>Pilih Hari</ion-label>
          <ion-select v-model="selectedDay" interface="alert" placeholder="Pilih hari">
            
            <ion-select-option
              v-for="date in availableDays"
              :key="date"
              :value="date"
            >
              {{ formatTanggal(date) }}
            </ion-select-option>
          </ion-select>
        </ion-item>

        <p class="judul-list"> Prakiraan cuaca {{ selectedDay ? 'pada ' + formatTanggal(selectedDay) : '' }} </p>

        <div
          v-for="(item, idx) in dataTerpilih"
          :key="idx"
          class="baris-jam" >

          <span class="jam">{{ formatJam(item.time) }}</span>

          <div class="bar-wrap">
            <div
              class="bar-isi"
              :style="{ width: getTinggiBar(item.temp) + '%' }"
            ></div>
          </div>
          <span class="suhu-item">{{ item.temp }}°C</span>
        </div>

      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>

import { ref, computed, onMounted } from 'vue'
import axios from 'axios'
import {
  IonPage, IonHeader, IonToolbar, IonTitle,
  IonContent, IonSpinner, IonButton,
  IonItem, IonLabel, IonSelect, IonSelectOption
} from '@ionic/vue'

const loading = ref(true)
const error   = ref(null)

const semuaData = ref([])
const selectedDay = ref('')

const availableDays = computed(() => {
  const dates = []
  semuaData.value.forEach(item => {
    const date = item.time.split('T')[0]
    if (!dates.includes(date)) dates.push(date)
  })
  return dates
})

const hariTerpilih = computed(() => selectedDay.value || availableDays.value[0] || '')

async function ambilData() {
  loading.value = true  
  error.value   = null  
  try {
    const respons = await axios.get('https://api.open-meteo.com/v1/forecast', {
      params: {
        latitude: -6.2,
        longitude: 106.8,
        hourly: 'temperature_2m'
      }
    })

    const json = respons.data

    semuaData.value = json.hourly.time.map((waktu, i) => ({
      time: waktu,
      temp: json.hourly.temperature_2m[i]
    }))

  } catch {
    error.value = 'Tidak dapat terhubung ke server.'
  } finally {
    loading.value = false
  }
}

const dataTerpilih = computed(() =>
  semuaData.value.filter(d => d.time.startsWith(hariTerpilih.value))
)

const suhuSekarang = computed(() => {
  if (!hariTerpilih.value) return '--'
  const now = new Date()
  const hariSekarang = now.toISOString().split('T')[0]
  const jamSekarang = now.toISOString().slice(0, 13) 

  if (hariTerpilih.value === hariSekarang) {
    const cocok = semuaData.value.find(d => d.time.startsWith(jamSekarang))
    return cocok ? cocok.temp : (dataTerpilih.value[0]?.temp ?? '--')
  }

  return dataTerpilih.value[0]?.temp ?? '--'
})

const minSuhu = computed(() =>
  dataTerpilih.value.length
    ? Math.min(...dataTerpilih.value.map(d => d.temp))
    : '--'
)

const maxSuhu = computed(() =>
  dataTerpilih.value.length
    ? Math.max(...dataTerpilih.value.map(d => d.temp))
    : '--'
)

const rataRata = computed(() => {
  if (!dataTerpilih.value.length) return '--'
  const total = dataTerpilih.value.reduce((acc, d) => acc + d.temp, 0)
  return (total / dataTerpilih.value.length).toFixed(1)
})

function formatJam(waktu) {
  return waktu.split('T')[1].slice(0, 5)
}

function formatTanggal(dateString) {
  return new Date(dateString).toLocaleDateString('id-ID', {
    weekday: 'long',
    day: 'numeric',
    month: 'long'
  })
}

function getTinggiBar(suhu) {
  return Math.min(100, Math.max(5, ((suhu - 20) / 15) * 100))
}

onMounted(ambilData)
</script>

<style scoped>

ion-toolbar {
  --background: #767474;
  --color: #ffffff;
}
ion-title { font-size: 16px; font-weight: 600; letter-spacing: 0.3px; }

ion-content { --background: #ffffff; }

.konten { padding: 30px; }
.pilih-hari {
  margin-bottom: 19px;
}

.kartu-utama {
  background: #1792df;          
   border-radius: 100px;
  padding: 28px 24px 22px;
  margin-bottom: 10px;
  text-align: center;
}
.angka-besar {
  font-size: 72px;
  font-weight: 200;              
  color: #ffff00;
  line-height: 1;
  letter-spacing: -2px;
}

.baris-stat {
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-top: 18px;
}
.stat { display: flex; flex-direction: column; align-items: center; gap: 4px; }
.stat-angka { font-size: 18px; font-weight: 500; color: #ffe600; }
.stat-label { font-size: 11px; color: #ffffff; text-transform: uppercase; letter-spacing: 0.5px; }

.judul-list {
  font-size: 13px;
  color: #040404;
  margin: 0 0 12px;
  letter-spacing: 0.3px;
}

.baris-jam {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 10px 0;
  border-bottom: 1px solid #000000;
}

.jam {
  width: 44px;
  font-size: 13px;
  color: #000000;
  flex-shrink: 0;              
}

.bar-wrap {
  flex: 1;                      
  height: 4px;
  background: #565758;
  border-radius: 999px;
  overflow: hidden;
}

.bar-isi {
  height: 100%;
  background: #ecfd00;         
  border-radius: 999px;
  transition: width 0.4s ease;  
}

.suhu-item {
  width: 52px;
  text-align: right;
  font-size: 14px;
  font-weight: 500;
  color: #000000;
  flex-shrink: 0;
}
</style>

