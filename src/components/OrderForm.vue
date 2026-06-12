<template>
  <div class="order-form-container card">
    <div class="section-header" style="margin-bottom: 1.5rem;">
      <h2 style="font-size: 1.4rem;"><i class="fas fa-shipping-fast" style="color: var(--primary-navy);"></i> Buat Delivery Order Baru</h2>
      <p style="font-size: 0.85rem;">Input data pengiriman pesanan bahan ajar mahasiswa Universitas Terbuka</p>
    </div>

    <div class="form-grid">
      <!-- Auto Generated DO Number -->
      <div class="form-group">
        <label>Nomor DO (Otomatis)</label>
        <input 
          type="text" 
          :value="generatedDoNumber" 
          class="form-control" 
          disabled 
          style="font-family: monospace; font-weight: bold; background-color: #f1f5f9; color: var(--primary-navy-light);"
        />
      </div>

      <!-- NIM Input -->
      <div class="form-group">
        <label>NIM Mahasiswa</label>
        <input 
          type="text" 
          v-model="form.nim" 
          class="form-control" 
          :class="{ invalid: errors.nim }"
          placeholder="Contoh: 123456789"
          maxlength="10"
        />
        <span class="error-text" v-if="errors.nim">{{ errors.nim }}</span>
      </div>

      <!-- Nama Input -->
      <div class="form-group">
        <label>Nama Mahasiswa</label>
        <input 
          type="text" 
          v-model="form.nama" 
          class="form-control" 
          :class="{ invalid: errors.nama }"
          placeholder="Nama Lengkap"
        />
        <span class="error-text" v-if="errors.nama">{{ errors.nama }}</span>
      </div>

      <!-- Ekspedisi Selection -->
      <div class="form-group">
        <label>Layanan Ekspedisi</label>
        <select 
          v-model="form.ekspedisi" 
          class="form-control" 
          :class="{ invalid: errors.ekspedisi }"
        >
          <option value="">Pilih Layanan</option>
          <option v-for="exp in formattedExpeditions" :key="exp.kode" :value="exp.nama">
            {{ exp.nama }}
          </option>
        </select>
        <span class="error-text" v-if="errors.ekspedisi">{{ errors.ekspedisi }}</span>
      </div>

      <!-- Paket Bahan Ajar Selection -->
      <div class="form-group">
        <label>Paket Bahan Ajar</label>
        <select 
          v-model="form.paketKode" 
          class="form-control" 
          :class="{ invalid: errors.paketKode }"
        >
          <option value="">Pilih Paket</option>
          <option v-for="p in paket" :key="p.kode" :value="p.kode">
            {{ p.kode }} - {{ p.nama }}
          </option>
        </select>
        <span class="error-text" v-if="errors.paketKode">{{ errors.paketKode }}</span>
      </div>

      <!-- Tanggal Kirim -->
      <div class="form-group">
        <label>Tanggal Kirim</label>
        <input 
          type="date" 
          v-model="form.tanggalKirimRaw" 
          class="form-control" 
          :class="{ invalid: errors.tanggalKirim }"
        />
        <span class="error-text" v-if="errors.tanggalKirim">{{ errors.tanggalKirim }}</span>
      </div>

      <!-- Total Harga (Rp Auto filled) -->
      <div class="form-group">
        <label>Total Harga (Otomatis)</label>
        <input 
          type="text" 
          :value="formattedTotalHarga" 
          class="form-control" 
          disabled 
          style="font-weight: 700; color: var(--text-dark); background-color: #f1f5f9;"
        />
      </div>
    </div>

    <!-- Package Content Preview (Triggers dynamically when package is selected) -->
    <div class="paket-details-card" v-if="selectedPaketDetails">
      <h4><i class="fas fa-box-open"></i> Isi Paket Bahan Ajar: {{ selectedPaketDetails.nama }}</h4>
      <div style="margin-top: 0.5rem;">
        <ul>
          <li v-for="item in selectedPaketDetails.isi" :key="item">
            <i class="fas fa-caret-right"></i> {{ item }} - {{ getBookTitle(item) }}
          </li>
        </ul>
      </div>
    </div>

    <div style="display: flex; gap: 0.5rem; justify-content: flex-end; margin-top: 1.5rem;">
      <button class="btn btn-outline" @click="resetForm">
        <i class="fas fa-undo"></i> Reset
      </button>
      <button class="btn btn-primary" @click="submitOrder">
        <i class="fas fa-paper-plane"></i> Buat Pesanan
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'OrderForm',
  props: {
    paket: {
      type: Array,
      required: true
    },
    pengirimanList: {
      type: Array,
      required: true
    },
    trackingList: {
      type: Array,
      required: true
    },
    stocks: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      form: {
        nim: '',
        nama: '',
        ekspedisi: '',
        paketKode: '',
        tanggalKirimRaw: new Date().toISOString().substr(0, 10), // defaults to today
        total: 0
      },
      errors: {}
    };
  },
  watch: {
    // Watcher: automatically set the price and print items preview when package selection changes
    'form.paketKode'(newVal) {
      if (newVal) {
        const selected = this.paket.find(p => p.kode === newVal);
        if (selected) {
          this.form.total = selected.harga;
          return;
        }
      }
      this.form.total = 0;
    }
  },
  computed: {
    formattedExpeditions() {
      // Map pengirimanList (Reguler/Ekspres) to JNE services as required by task
      return this.pengirimanList.map(item => {
        const prefix = "JNE ";
        const shortName = item.nama.includes("Ekspres") ? "Express" : "Regular";
        return {
          kode: item.kode,
          nama: `${prefix}${shortName} (${item.nama})`
        };
      });
    },
    generatedDoNumber() {
      const currentYear = new Date().getFullYear();
      let maxSeq = 0;
      
      // Calculate sequence by scanning existing tracking entries
      this.trackingList.forEach(entry => {
        // Entry is an object where keys are DO IDs, e.g. { "DO2025-0001": {...} }
        const keys = Object.keys(entry);
        if (keys.length > 0) {
          const doId = keys[0];
          // Match DO[Year]-[Sequence]
          const match = doId.match(/DO(\d{4})-(\d+)/);
          if (match && parseInt(match[1]) === currentYear) {
            const seq = parseInt(match[2]);
            if (seq > maxSeq) {
              maxSeq = seq;
            }
          }
        }
      });
      
      const nextSeq = maxSeq + 1;
      // Pad to 3 digits as requested by prompt (e.g. 001) or match length of 4 digits if needed
      // Instruction says: "DO2025-001 untuk data awal, DO2025-002 dst."
      const paddedSeq = String(nextSeq).padStart(3, '0');
      return `DO${currentYear}-${paddedSeq}`;
    },
    selectedPaketDetails() {
      if (!this.form.paketKode) return null;
      return this.paket.find(p => p.kode === this.form.paketKode) || null;
    },
    formattedTotalHarga() {
      return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
      }).format(this.form.total);
    }
  },
  methods: {
    getBookTitle(kode) {
      const book = this.stocks.find(s => s.kode === kode);
      return book ? book.judul : 'Bahan Ajar UT';
    },
    formatIndonesianDate(dateString) {
      if (!dateString) return '';
      const months = [
        'Januari', 'Februari', 'Maret', 'April', 'Mei', 'Juni',
        'Juli', 'Agustus', 'September', 'Oktober', 'November', 'Desember'
      ];
      const d = new Date(dateString);
      const day = d.getDate();
      const month = months[d.getMonth()];
      const year = d.getFullYear();
      return `${day} ${month} ${year}`;
    },
    validateForm() {
      this.errors = {};
      
      if (!this.form.nim.trim()) {
        this.errors.nim = 'NIM wajib diisi';
      } else if (!/^\d{9,10}$/.test(this.form.nim.trim())) {
        this.errors.nim = 'NIM harus berisi 9-10 digit angka';
      }

      if (!this.form.nama.trim()) {
        this.errors.nama = 'Nama lengkap wajib diisi';
      }

      if (!this.form.ekspedisi) {
        this.errors.ekspedisi = 'Ekspedisi wajib dipilih';
      }

      if (!this.form.paketKode) {
        this.errors.paketKode = 'Paket bahan ajar wajib dipilih';
      }

      if (!this.form.tanggalKirimRaw) {
        this.errors.tanggalKirim = 'Tanggal kirim wajib diisi';
      }

      return Object.keys(this.errors).length === 0;
    },
    submitOrder() {
      if (!this.validateForm()) return;

      const newOrder = {
        doNumber: this.generatedDoNumber,
        nim: this.form.nim.trim(),
        nama: this.form.nama.trim(),
        status: 'Penerimaan di Loket',
        ekspedisi: this.form.ekspedisi,
        tanggalKirim: this.formatIndonesianDate(this.form.tanggalKirimRaw),
        paket: this.form.paketKode,
        total: this.form.total,
        perjalanan: [
          {
            waktu: new Date().toISOString().replace('T', ' ').substring(0, 19),
            keterangan: `Penerimaan di Loket Ekspedisi: ${this.form.ekspedisi.split(' ')[0]}`
          }
        ]
      };

      this.$emit('submit-order', newOrder);
      this.resetForm();
    },
    resetForm() {
      this.form = {
        nim: '',
        nama: '',
        ekspedisi: '',
        paketKode: '',
        tanggalKirimRaw: new Date().toISOString().substr(0, 10),
        total: 0
      };
      this.errors = {};
    }
  }
};
</script>
