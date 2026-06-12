<template>
  <div class="stock-table-container">
    <!-- Mini CRUD: Tambah / Edit Stock Form -->
    <div class="mini-crud-container card" v-if="showForm">
      <div class="mini-crud-header">
        <i :class="isEditMode ? 'fas fa-edit' : 'fas fa-plus-circle'"></i>
        {{ isEditMode ? 'Perbarui Stok Bahan Ajar' : 'Tambah Stok Bahan Ajar Baru' }}
      </div>
      
      <div class="form-grid">
        <div class="form-group">
          <label>Kode MK</label>
          <input 
            type="text" 
            v-model="form.kode" 
            class="form-control" 
            :class="{ invalid: errors.kode }"
            :disabled="isEditMode"
            placeholder="Contoh: EKMA4116"
            @keyup.enter="saveStock"
          />
          <span class="error-text" v-if="errors.kode">{{ errors.kode }}</span>
        </div>
        
        <div class="form-group">
          <label>Judul Mata Kuliah</label>
          <input 
            type="text" 
            v-model="form.judul" 
            class="form-control" 
            :class="{ invalid: errors.judul }"
            placeholder="Nama lengkap MK"
            @keyup.enter="saveStock"
          />
          <span class="error-text" v-if="errors.judul">{{ errors.judul }}</span>
        </div>
        
        <div class="form-group">
          <label>Kategori</label>
          <select 
            v-model="form.kategori" 
            class="form-control" 
            :class="{ invalid: errors.kategori }"
            @keyup.enter="saveStock"
          >
            <option value="">Pilih Kategori</option>
            <option v-for="cat in kategoriList" :key="cat" :value="cat">{{ cat }}</option>
          </select>
          <span class="error-text" v-if="errors.kategori">{{ errors.kategori }}</span>
        </div>
        
        <div class="form-group">
          <label>UT-Daerah (UPBJJ)</label>
          <select 
            v-model="form.upbjj" 
            class="form-control" 
            :class="{ invalid: errors.upbjj }"
            @keyup.enter="saveStock"
          >
            <option value="">Pilih UT Daerah</option>
            <option v-for="up in upbjjList" :key="up" :value="up">{{ up }}</option>
          </select>
          <span class="error-text" v-if="errors.upbjj">{{ errors.upbjj }}</span>
        </div>
        
        <div class="form-group">
          <label>Lokasi Rak</label>
          <input 
            type="text" 
            v-model="form.lokasiRak" 
            class="form-control" 
            :class="{ invalid: errors.lokasiRak }"
            placeholder="Contoh: R1-A3"
            @keyup.enter="saveStock"
          />
          <span class="error-text" v-if="errors.lokasiRak">{{ errors.lokasiRak }}</span>
        </div>
        
        <div class="form-group">
          <label>Harga (Rp)</label>
          <input 
            type="number" 
            v-model.number="form.harga" 
            class="form-control" 
            :class="{ invalid: errors.harga }"
            placeholder="Contoh: 65000"
            @keyup.enter="saveStock"
          />
          <span class="error-text" v-if="errors.harga">{{ errors.harga }}</span>
        </div>
        
        <div class="form-group">
          <label>Qty Stok</label>
          <input 
            type="number" 
            v-model.number="form.qty" 
            class="form-control" 
            :class="{ invalid: errors.qty }"
            placeholder="0"
            @keyup.enter="saveStock"
          />
          <span class="error-text" v-if="errors.qty">{{ errors.qty }}</span>
        </div>
        
        <div class="form-group">
          <label>Safety Stock</label>
          <input 
            type="number" 
            v-model.number="form.safety" 
            class="form-control" 
            :class="{ invalid: errors.safety }"
            placeholder="10"
            @keyup.enter="saveStock"
          />
          <span class="error-text" v-if="errors.safety">{{ errors.safety }}</span>
        </div>
      </div>
      
      <div class="form-group">
        <label>Catatan Tambahan (HTML)</label>
        <input 
          type="text" 
          v-model="form.catatanHTML" 
          class="form-control" 
          placeholder="Contoh: <strong>Cetak baru</strong>"
          @keyup.enter="saveStock"
        />
      </div>

      <div style="display: flex; gap: 0.5rem; justify-content: flex-end; margin-top: 1rem;">
        <button class="btn btn-outline btn-sm" @click="cancelForm">Batal</button>
        <button class="btn btn-primary btn-sm" @click="saveStock">
          <i class="fas fa-save"></i> {{ isEditMode ? 'Perbarui' : 'Simpan' }}
        </button>
      </div>
    </div>

    <!-- Toolbar Filters & Sorting -->
    <div class="card" style="padding: 1.25rem; margin-bottom: 1.5rem;">
      <div class="toolbar-section">
        <!-- Search bar -->
        <div class="search-box">
          <i class="fas fa-search"></i>
          <input 
            type="text" 
            v-model="filters.search" 
            placeholder="Cari Kode atau Judul Bahan Ajar..." 
          />
        </div>

        <div style="display: flex; gap: 0.5rem;">
          <button 
            v-if="!showForm" 
            class="btn btn-primary btn-sm" 
            @click="openAddForm"
          >
            <i class="fas fa-plus"></i> Tambah Stok
          </button>
          <button class="btn btn-outline btn-sm" @click="resetFilters">
            <i class="fas fa-undo-alt"></i> Reset Filter
          </button>
        </div>
      </div>

      <div class="filter-group">
        <!-- Filter UT-Daerah -->
        <div class="select-custom">
          <select v-model="filters.upbjj">
            <option value="">Semua UT-Daerah</option>
            <option v-for="up in upbjjList" :key="up" :value="up">{{ up }}</option>
          </select>
        </div>

        <!-- Dependent option: Kategori selection only shows when UT-Daerah filter is selected -->
        <div class="select-custom" v-if="filters.upbjj">
          <select v-model="filters.kategori">
            <option value="">Semua Kategori</option>
            <option v-for="cat in kategoriList" :key="cat" :value="cat">{{ cat }}</option>
          </select>
        </div>

        <!-- Low stock critical alert checkbox -->
        <label class="checkbox-custom">
          <input type="checkbox" v-model="filters.criticalOnly" />
          <span><i class="fas fa-exclamation-triangle" style="color: var(--state-warning);"></i> Butuh Reorder (Stok &lt; Safety / Kosong)</span>
        </label>
      </div>
    </div>

    <!-- Main Stock Table -->
    <div class="table-responsive card" style="padding: 0;">
      <table class="sitta-table">
        <thead>
          <tr>
            <th class="sortable" @click="toggleSort('kode')">
              Kode / Judul MK
              <i :class="getSortIcon('kode')"></i>
            </th>
            <th>Kategori</th>
            <th>UT-Daerah</th>
            <th>Rak</th>
            <th class="sortable" @click="toggleSort('harga')" style="text-align: right;">
              Harga
              <i :class="getSortIcon('harga')"></i>
            </th>
            <th class="sortable" @click="toggleSort('qty')" style="text-align: right;">
              Stok
              <i :class="getSortIcon('qty')"></i>
            </th>
            <th style="text-align: right;">Safety</th>
            <th style="text-align: center;">Status</th>
            <th style="text-align: center;">Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="filteredStocks.length === 0">
            <td colspan="9" style="text-align: center; padding: 2rem; color: var(--text-muted);">
              <i class="fas fa-folder-open" style="font-size: 2rem; margin-bottom: 0.5rem; display: block;"></i>
              Tidak ada data stok bahan ajar yang ditemukan
            </td>
          </tr>
          <tr v-for="stock in filteredStocks" :key="stock.kode">
            <td>
              <div class="item-title-wrapper">
                <span class="item-code">{{ stock.kode }}</span>
                <span class="item-title">{{ stock.judul }}</span>
              </div>
            </td>
            <td>{{ stock.kategori }}</td>
            <td>{{ stock.upbjj }}</td>
            <td><code style="font-size: 0.8rem; font-weight: 700;">{{ stock.lokasiRak }}</code></td>
            <td style="text-align: right;" class="item-price">{{ formatRupiah(stock.harga) }}</td>
            <td style="text-align: right; font-weight: 700;" :style="{ color: stock.qty === 0 ? 'var(--state-danger)' : 'inherit' }">
              {{ formatQty(stock.qty) }}
            </td>
            <td style="text-align: right; color: var(--text-muted);">{{ formatQty(stock.safety) }}</td>
            <td style="text-align: center;">
              <!-- Embedded Status Badge with Note hover Tooltip -->
              <StatusBadge 
                :qty="stock.qty" 
                :safety="stock.safety" 
                :catatan-html="stock.catatanHTML" 
              />
            </td>
            <td style="text-align: center;">
              <div style="display: flex; gap: 0.5rem; justify-content: center;">
                <button class="btn btn-outline btn-sm" style="padding: 0.3rem 0.5rem;" @click="editStock(stock)" title="Edit Data">
                  <i class="fas fa-pencil-alt" style="color: var(--primary-navy-light);"></i>
                </button>
                <button class="btn btn-outline btn-sm btn-danger-hover" style="padding: 0.3rem 0.5rem;" @click="confirmDelete(stock.kode)" title="Hapus Data">
                  <i class="fas fa-trash-alt" style="color: var(--state-danger);"></i>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Reusable Confirmation Dialog for deletion -->
    <AppModal :show="deleteModal.show" title="Konfirmasi Penghapusan" @close="deleteModal.show = false">
      <div style="text-align: center;">
        <i class="fas fa-exclamation-triangle" style="font-size: 3rem; color: var(--state-danger); margin-bottom: 1rem;"></i>
        <p style="font-weight: 600; color: var(--text-dark); font-size: 1.1rem; margin-bottom: 0.5rem;">Apakah Anda Yakin?</p>
        <p style="color: var(--text-muted);">
          Anda akan menghapus data stok untuk kode bahan ajar <strong>{{ deleteModal.kode }}</strong>. Aksi ini tidak dapat dibatalkan.
        </p>
      </div>
      <template #footer>
        <button class="btn btn-outline btn-sm" @click="deleteModal.show = false">Batal</button>
        <button class="btn btn-danger btn-sm" @click="executeDelete">Hapus Sekarang</button>
      </template>
    </AppModal>
  </div>
</template>

<script>
import StatusBadge from './StatusBadge.vue';
import AppModal from './AppModal.vue';

export default {
  name: 'StockTable',
  components: {
    StatusBadge,
    AppModal
  },
  props: {
    stocks: {
      type: Array,
      required: true
    },
    upbjjList: {
      type: Array,
      required: true
    },
    kategoriList: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      showForm: false,
      isEditMode: false,
      form: {
        kode: '',
        judul: '',
        kategori: '',
        upbjj: '',
        lokasiRak: '',
        harga: 0,
        qty: 0,
        safety: 10,
        catatanHTML: ''
      },
      errors: {},
      filters: {
        search: '',
        upbjj: '',
        kategori: '',
        criticalOnly: false
      },
      sorting: {
        column: 'judul', // default
        dir: 'asc'
      },
      deleteModal: {
        show: false,
        kode: ''
      }
    };
  },
  // Watch active filters to demonstrate Watchers requirement
  watch: {
    'filters.upbjj'(newVal) {
      // Dependent Filter Option logic
      // If UT-Daerah selection is cleared, reset the Kategori filter selection
      if (!newVal) {
        this.filters.kategori = '';
      }
    }
  },
  computed: {
    // Computed property ensures filters are cached and do NOT recompute unless states change!
    filteredStocks() {
      let result = [...this.stocks];

      // 1. Text Search Filter (match by code or title case-insensitive)
      if (this.filters.search.trim()) {
        const query = this.filters.search.toLowerCase().trim();
        result = result.filter(s => 
          s.kode.toLowerCase().includes(query) || 
          s.judul.toLowerCase().includes(query)
        );
      }

      // 2. UT-Daerah filter
      if (this.filters.upbjj) {
        result = result.filter(s => s.upbjj === this.filters.upbjj);
      }

      // 3. Dependent Kategori filter (only valid if UPBJJ is selected)
      if (this.filters.upbjj && this.filters.kategori) {
        result = result.filter(s => s.kategori === this.filters.kategori);
      }

      // 4. Critical Stock Alert (qty < safety or qty == 0)
      if (this.filters.criticalOnly) {
        result = result.filter(s => s.qty === 0 || s.qty < s.safety);
      }

      // 5. Sorting
      const col = this.sorting.column;
      const dir = this.sorting.dir === 'asc' ? 1 : -1;

      result.sort((a, b) => {
        let valA = a[col];
        let valB = b[col];

        // Format case-insensitive sorting for strings
        if (typeof valA === 'string') {
          valA = valA.toLowerCase();
          valB = valB.toLowerCase();
        }

        if (valA < valB) return -1 * dir;
        if (valA > valB) return 1 * dir;
        return 0;
      });

      return result;
    }
  },
  methods: {
    formatRupiah(num) {
      return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
      }).format(num);
    },
    formatQty(qty) {
      return `${qty} buah`;
    },
    toggleSort(column) {
      if (this.sorting.column === column) {
        this.sorting.dir = this.sorting.dir === 'asc' ? 'desc' : 'asc';
      } else {
        this.sorting.column = column;
        this.sorting.dir = 'asc';
      }
    },
    getSortIcon(column) {
      if (this.sorting.column !== column) return 'fas fa-sort text-light';
      return this.sorting.dir === 'asc' ? 'fas fa-sort-up' : 'fas fa-sort-down';
    },
    resetFilters() {
      this.filters.search = '';
      this.filters.upbjj = '';
      this.filters.kategori = '';
      this.filters.criticalOnly = false;
    },
    openAddForm() {
      this.isEditMode = false;
      this.form = {
        kode: '',
        judul: '',
        kategori: '',
        upbjj: '',
        lokasiRak: '',
        harga: 0,
        qty: 0,
        safety: 10,
        catatanHTML: ''
      };
      this.errors = {};
      this.showForm = true;
    },
    editStock(stock) {
      this.isEditMode = true;
      this.form = { ...stock };
      this.errors = {};
      this.showForm = true;
    },
    cancelForm() {
      this.showForm = false;
      this.errors = {};
    },
    validateForm() {
      this.errors = {};
      
      if (!this.form.kode.trim()) {
        this.errors.kode = 'Kode MK wajib diisi';
      } else if (!this.isEditMode) {
        // Unique validation on create
        const exists = this.stocks.some(s => s.kode.toUpperCase() === this.form.kode.toUpperCase());
        if (exists) this.errors.kode = 'Kode MK sudah terdaftar';
      }

      if (!this.form.judul.trim()) this.errors.judul = 'Judul MK wajib diisi';
      if (!this.form.kategori) this.errors.kategori = 'Kategori wajib dipilih';
      if (!this.form.upbjj) this.errors.upbjj = 'UT-Daerah wajib dipilih';
      if (!this.form.lokasiRak.trim()) this.errors.lokasiRak = 'Lokasi rak wajib diisi';
      
      if (typeof this.form.harga !== 'number' || this.form.harga < 0) {
        this.errors.harga = 'Harga harus angka positif';
      }
      
      if (typeof this.form.qty !== 'number' || this.form.qty < 0) {
        this.errors.qty = 'Qty stok harus angka positif';
      }

      if (typeof this.form.safety !== 'number' || this.form.safety < 0) {
        this.errors.safety = 'Safety stock harus angka positif';
      }

      return Object.keys(this.errors).length === 0;
    },
    saveStock() {
      if (!this.validateForm()) return;

      this.form.kode = this.form.kode.toUpperCase().trim();
      this.form.judul = this.form.judul.trim();
      this.form.lokasiRak = this.form.lokasiRak.toUpperCase().trim();

      if (this.isEditMode) {
        // Emit update event to parent
        this.$emit('update-stock', { ...this.form });
      } else {
        // Emit create event to parent
        this.$emit('create-stock', { ...this.form });
      }

      this.showForm = false;
      this.errors = {};
    },
    confirmDelete(kode) {
      this.deleteModal.kode = kode;
      this.deleteModal.show = true;
    },
    executeDelete() {
      this.$emit('delete-stock', this.deleteModal.kode);
      this.deleteModal.show = false;
    }
  }
};
</script>

<style scoped>
.btn-danger-hover:hover {
  background-color: var(--state-danger-bg);
  border-color: rgba(239, 68, 68, 0.4);
}
</style>
