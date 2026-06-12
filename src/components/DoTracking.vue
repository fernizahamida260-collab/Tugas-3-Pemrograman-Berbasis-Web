<template>
  <div class="do-tracking-container">
    <div class="card" style="padding: 1.75rem;">
      <div class="section-header" style="margin-bottom: 1.5rem;">
        <h2><i class="fas fa-search-location" style="color: var(--primary-navy);"></i> Pelacakan Pengiriman (DO)</h2>
        <p>Lacak pengiriman bahan ajar Anda dengan Nomor Delivery Order (DO) atau Nomor Induk Mahasiswa (NIM)</p>
      </div>

      <!-- Search Input with Keyboard event handlers -->
      <div style="display: flex; gap: 1rem; align-items: stretch;">
        <div class="search-box" style="max-width: none;">
          <i class="fas fa-search"></i>
          <input 
            type="text" 
            v-model="searchQuery" 
            placeholder="Masukkan Nomor DO (misal: DO2025-0001) atau NIM... Tekan Enter untuk mencari" 
            @keyup.enter="searchDO"
            @keyup.esc="clearSearch"
          />
        </div>
        <button class="btn btn-primary" @click="searchDO">
          <i class="fas fa-search"></i> Cari
        </button>
        <button class="btn btn-outline" @click="clearSearch">
          Batal (Esc)
        </button>
      </div>
    </div>

    <div class="quick-search-info" v-if="searchPerformed">
      <span>
        Hasil Pencarian untuk: <strong>"{{ lastSearchQuery }}"</strong>
      </span>
      <span>
        Ditemukan: <strong>{{ searchResults.length }} data DO</strong>
      </span>
    </div>

    <!-- Multiple Results Selector (If matching NIM has multiple DOs) -->
    <div class="card" v-if="searchResults.length > 1" style="padding: 1.25rem; margin-bottom: 1.5rem;">
      <h3 style="font-size: 1rem; font-weight: 700; margin-bottom: 0.75rem; color: var(--primary-navy-light);">
        <i class="fas fa-list-ol"></i> Pilih Pesanan DO yang ingin dilacak:
      </h3>
      <div style="display: flex; flex-direction: column; gap: 0.5rem;">
        <button 
          v-for="res in searchResults" 
          :key="res.id" 
          class="btn btn-outline btn-sm"
          style="justify-content: space-between; text-align: left; width: 100%;"
          :class="{ 'btn-accent': activeDO && activeDO.id === res.id }"
          @click="selectDO(res)"
        >
          <span>
            <strong style="font-family: monospace;">{{ res.id }}</strong> - {{ res.nama }} ({{ res.ekspedisi }})
          </span>
          <span>
            Tanggal: {{ res.tanggalKirim }} | Status: <strong>{{ res.status }}</strong>
          </span>
        </button>
      </div>
    </div>

    <!-- No results display -->
    <div class="card" v-if="searchPerformed && searchResults.length === 0" style="text-align: center; padding: 3rem;">
      <i class="fas fa-box-open" style="font-size: 4rem; color: var(--text-light); margin-bottom: 1rem;"></i>
      <p style="font-size: 1.2rem; font-weight: 700; color: var(--text-dark);">Pencarian Tidak Ditemukan</p>
      <p style="color: var(--text-muted); font-size: 0.95rem; margin-top: 0.25rem;">
        Silakan periksa kembali Nomor DO atau NIM yang Anda masukkan.
      </p>
    </div>

    <!-- Active DO Details and Timeline View -->
    <div class="grid-details-timeline" v-if="activeDO">
      <div class="card" style="padding: 1.5rem;">
        <h3 style="font-size: 1.15rem; font-weight: 800; border-bottom: 2px solid var(--border-light); padding-bottom: 0.5rem; margin-bottom: 1rem; color: var(--primary-navy-light);">
          <i class="fas fa-info-circle"></i> Rincian Pengiriman
        </h3>
        
        <table class="details-table">
          <tbody>
            <tr>
              <th>Nomor DO</th>
              <td style="font-family: monospace; font-weight: 700; color: var(--primary-navy);">{{ activeDO.id }}</td>
            </tr>
            <tr>
              <th>NIM Mahasiswa</th>
              <td>{{ activeDO.nim }}</td>
            </tr>
            <tr>
              <th>Nama Lengkap</th>
              <td style="font-weight: 600;">{{ activeDO.nama }}</td>
            </tr>
            <tr>
              <th>Layanan Kirim</th>
              <td>{{ activeDO.ekspedisi }}</td>
            </tr>
            <tr>
              <th>Paket Pesanan</th>
              <td>
                <span class="item-code" style="display: inline-block;">{{ activeDO.paket }}</span>
                <div style="font-size: 0.85rem; color: var(--text-muted); font-weight: 600; margin-top: 0.25rem;">
                  Isi: {{ getPaketIsi(activeDO.paket) }}
                </div>
              </td>
            </tr>
            <tr>
              <th>Tanggal Kirim</th>
              <td style="font-weight: 600;">{{ activeDO.tanggalKirim }}</td>
            </tr>
            <tr>
              <th>Total Harga</th>
              <td style="font-weight: 800; color: var(--primary-navy);">{{ formatRupiah(activeDO.total) }}</td>
            </tr>
            <tr>
              <th>Status Terkini</th>
              <td>
                <span :class="['badge-status', getStatusClass(activeDO.status)]" style="cursor: default;">
                  {{ activeDO.status }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Timeline & Progress Adding Form -->
      <div class="card" style="padding: 1.5rem;">
        <h3 style="font-size: 1.15rem; font-weight: 800; border-bottom: 2px solid var(--border-light); padding-bottom: 0.5rem; margin-bottom: 1rem; color: var(--primary-navy-light);">
          <i class="fas fa-road"></i> Status Perjalanan Paket
        </h3>

        <!-- Add progress log form -->
        <div class="mini-crud-container" style="padding: 1rem; border-style: solid; margin-bottom: 1.5rem; background-color: var(--bg-main);">
          <div class="mini-crud-header" style="font-size: 0.9rem; margin-bottom: 0.5rem;">
            <i class="fas fa-plus-circle"></i> Tambah Progress Pengiriman
          </div>
          <div style="display: flex; gap: 0.5rem; flex-direction: column;">
            <input 
              type="text" 
              v-model="newProgressText" 
              class="form-control" 
              placeholder="Masukkan keterangan lokasi/kegiatan terbaru..."
              @keyup.enter="addProgress"
            />
            <div style="display: flex; gap: 0.5rem; justify-content: space-between; align-items: center; margin-top: 0.25rem;">
              <span style="font-size: 0.75rem; color: var(--text-muted); font-weight: 500;">
                Waktu: <i class="far fa-clock"></i> {{ currentLocalTime }} (Otomatis)
              </span>
              <button class="btn btn-primary btn-sm" @click="addProgress">
                <i class="fas fa-plus"></i> Tambah Status
              </button>
            </div>
          </div>
        </div>

        <!-- Vertical Timeline Display -->
        <div class="timeline">
          <div class="timeline-item" v-for="(log, idx) in activeDO.perjalanan" :key="idx">
            <div class="timeline-badge"></div>
            <div class="timeline-content">
              <div class="timeline-time">
                <i class="far fa-clock"></i> {{ log.waktu }}
              </div>
              <div class="timeline-desc">{{ log.keterangan }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DoTracking',
  props: {
    trackingList: {
      type: Array,
      required: true
    },
    paket: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      searchQuery: '',
      lastSearchQuery: '',
      searchPerformed: false,
      searchResults: [],
      activeDO: null,
      newProgressText: '',
      currentLocalTime: ''
    };
  },
  mounted() {
    this.updateClock();
    // Keep dynamic clock ticking
    this.timer = setInterval(this.updateClock, 1000);
  },
  beforeUnmount() {
    clearInterval(this.timer);
  },
  watch: {
    searchQuery(newVal) {
      if (!newVal) {
        this.searchPerformed = false;
        this.searchResults = [];
        this.activeDO = null;
      }
    }
  },
  methods: {
    updateClock() {
      const now = new Date();
      const pad = (n) => String(n).padStart(2, '0');
      
      const date = `${now.getFullYear()}-${pad(now.getMonth()+1)}-${pad(now.getDate())}`;
      const time = `${pad(now.getHours())}:${pad(now.getMinutes())}:${pad(now.getSeconds())}`;
      
      this.currentLocalTime = `${date} ${time}`;
    },
    formatRupiah(num) {
      return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
      }).format(num);
    },
    getPaketIsi(paketKode) {
      const p = this.paket.find(item => item.kode === paketKode);
      return p ? p.isi.join(', ') : 'Bahan Ajar';
    },
    getStatusClass(status) {
      if (status.toLowerCase().includes('tiba') || status.toLowerCase().includes('selesai') || status.toLowerCase().includes('diterima')) {
        return 'badge-aman';
      }
      if (status.toLowerCase().includes('perjalanan') || status.toLowerCase().includes('transit')) {
        return 'badge-menipis';
      }
      return 'badge-kosong';
    },
    searchDO() {
      const query = this.searchQuery.trim().toUpperCase();
      this.lastSearchQuery = this.searchQuery.trim();
      this.searchResults = [];
      this.activeDO = null;

      if (!query) return;

      // Scan through array of objects in trackingList
      const matches = [];
      this.trackingList.forEach(entry => {
        // Entry matches { "DO2025-0001": { nim, nama, status, ... } }
        const doId = Object.keys(entry)[0];
        const details = entry[doId];

        if (doId.toUpperCase().includes(query) || details.nim.includes(query)) {
          matches.push({
            id: doId,
            ...details
          });
        }
      });

      this.searchResults = matches;
      this.searchPerformed = true;

      // If exactly one result is found, automatically select it!
      if (matches.length === 1) {
        this.selectDO(matches[0]);
      }
    },
    selectDO(res) {
      this.activeDO = res;
    },
    clearSearch() {
      this.searchQuery = '';
      this.lastSearchQuery = '';
      this.searchPerformed = false;
      this.searchResults = [];
      this.activeDO = null;
    },
    addProgress() {
      if (!this.newProgressText.trim() || !this.activeDO) return;

      const newLog = {
        waktu: this.currentLocalTime,
        keterangan: this.newProgressText.trim()
      };

      // Push to active local timeline
      this.activeDO.perjalanan.unshift(newLog); // prepend to show latest first or append as timeline?
      
      // Update overall DO status based on newest log
      this.activeDO.status = this.newProgressText.trim().substring(0, 30); // clip text for status label

      // Emit tracking list update to parent so data persists!
      this.$emit('update-tracking', {
        id: this.activeDO.id,
        updatedDO: {
          nim: this.activeDO.nim,
          nama: this.activeDO.nama,
          status: this.activeDO.status,
          ekspedisi: this.activeDO.ekspedisi,
          tanggalKirim: this.activeDO.tanggalKirim,
          paket: this.activeDO.paket,
          total: this.activeDO.total,
          perjalanan: this.activeDO.perjalanan
        }
      });

      this.newProgressText = '';
      console.log('Progress status successfully appended!');
    }
  }
};
</script>

<style scoped>
.grid-details-timeline {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.details-table {
  width: 100%;
  border-collapse: collapse;
}

.details-table th, .details-table td {
  padding: 0.75rem 0.5rem;
  border-bottom: 1px solid var(--border-light);
  font-size: 0.9rem;
}

.details-table th {
  text-align: left;
  color: var(--text-muted);
  font-weight: 700;
  width: 35%;
}

.details-table td {
  color: var(--text-dark);
}

.details-table tr:last-child th, .details-table tr:last-child td {
  border-bottom: none;
}

@media (max-width: 768px) {
  .grid-details-timeline {
    grid-template-columns: 1fr;
  }
}
</style>
