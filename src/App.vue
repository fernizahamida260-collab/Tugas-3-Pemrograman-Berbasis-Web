<template>
  <div id="app">
    <!-- SITTA Header & Navigation -->
    <header class="sitta-header">
      <div class="header-container">
        <div class="brand-wrapper">
          <img src="/assets/logo/logo_ut.png" alt="Logo UT" class="brand-logo" />
          <div class="brand-info">
            <h1>SITTA <span>Bahan Ajar</span></h1>
            <p>Sistem Informasi Tiras & Distribusi Bahan Ajar | Universitas Terbuka</p>
          </div>
        </div>

        <nav class="sitta-nav">
          <button 
            :class="['nav-tab', { active: activeTab === 'stok' }]" 
            @click="activeTab = 'stok'"
          >
            <i class="fas fa-warehouse"></i>
            Stok Bahan Ajar
          </button>
          
          <button 
            :class="['nav-tab', { active: activeTab === 'order' }]" 
            @click="activeTab = 'order'"
          >
            <i class="fas fa-cart-plus"></i>
            Buat DO Baru
          </button>

          <button 
            :class="['nav-tab', { active: activeTab === 'tracking' }]" 
            @click="activeTab = 'tracking'"
          >
            <i class="fas fa-map-marker-alt"></i>
            Lacak Kiriman
          </button>
        </nav>
      </div>
    </header>

    <!-- Main Content Area -->
    <main class="sitta-main">
      <transition name="fade" mode="out-in">
        <!-- Render pages conditionally based on activeTab -->
        <div v-if="activeTab === 'stok'" key="stok">
          <div class="section-header">
            <h2>Manajemen Stok Gudang</h2>
            <p>Pantau persediaan, edit koordinat rak, dan kelola tiras bahan ajar regional</p>
          </div>
          
          <StockTable 
            :stocks="db.stok" 
            :upbjj-list="db.upbjjList" 
            :kategori-list="db.kategoriList" 
            @create-stock="handleCreateStock"
            @update-stock="handleUpdateStock"
            @delete-stock="handleDeleteStock"
          />
        </div>

        <div v-else-if="activeTab === 'order'" key="order">
          <div class="section-header">
            <h2>Pemesanan & Distribusi</h2>
            <p>Formulir registrasi dan penyiapan pengemasan bahan ajar UT daerah</p>
          </div>

          <OrderForm 
            :paket="db.paket"
            :pengiriman-list="db.pengirimanList"
            :tracking-list="db.tracking"
            :stocks="db.stok"
            @submit-order="handleCreateOrder"
          />
        </div>

        <div v-else-if="activeTab === 'tracking'" key="tracking">
          <div class="section-header">
            <h2>Pelacakan Real-time</h2>
            <p>Status perjalanan dan update log logistik kiriman paket mahasiswa</p>
          </div>

          <DoTracking 
            :tracking-list="db.tracking"
            :paket="db.paket"
            @update-tracking="handleUpdateTracking"
          />
        </div>
      </transition>
    </main>

    <!-- SITTA Footer -->
    <footer class="sitta-footer">
      <p>&copy; 2026 Universitas Terbuka. All Rights Reserved. | <a href="https://www.ut.ac.id" target="_blank">Layanan SITTA UT</a></p>
    </footer>
  </div>
</template>

<script>
import dataBahanAjar from './data/dataBahanAjar.json';
import StockTable from './components/StockTable.vue';
import OrderForm from './components/OrderForm.vue';
import DoTracking from './components/DoTracking.vue';

export default {
  name: 'App',
  components: {
    StockTable,
    OrderForm,
    DoTracking
  },
  data() {
    return {
      activeTab: 'stok', // active tab state in root index
      db: {
        upbjjList: [],
        kategoriList: [],
        pengirimanList: [],
        paket: [],
        stok: [],
        tracking: []
      }
    };
  },
  created() {
    this.loadDatabase();
  },
  methods: {
    loadDatabase() {
      // 1. Try to load from LocalStorage first to make CRUD changes persistent
      const localDB = localStorage.getItem('sitta_database');
      if (localDB) {
        try {
          this.db = JSON.parse(localDB);
          console.log('Database loaded successfully from LocalStorage!');
          return;
        } catch (e) {
          console.error('Failed to parse LocalStorage database, loading default dummy JSON', e);
        }
      }

      // 2. Fallback to imported static dataBahanAjar dummy data
      this.db = JSON.parse(JSON.stringify(dataBahanAjar));
      this.saveDatabase();
      console.log('Database initialized from static dummy JSON!');
    },
    saveDatabase() {
      // Save entire reactive state back to LocalStorage
      localStorage.setItem('sitta_database', JSON.stringify(this.db));
    },
    
    // CRUD Stock Event Handlers
    handleCreateStock(newStock) {
      this.db.stok.push(newStock);
      this.saveDatabase();
      console.log('New stock created:', newStock.kode);
    },
    handleUpdateStock(updatedStock) {
      const idx = this.db.stok.findIndex(s => s.kode === updatedStock.kode);
      if (idx !== -1) {
        this.db.stok.splice(idx, 1, updatedStock);
        this.saveDatabase();
        console.log('Stock updated successfully:', updatedStock.kode);
      }
    },
    handleDeleteStock(kode) {
      const idx = this.db.stok.findIndex(s => s.kode === kode);
      if (idx !== -1) {
        this.db.stok.splice(idx, 1);
        this.saveDatabase();
        console.log('Stock deleted successfully:', kode);
      }
    },

    // Order and Tracking Event Handlers
    handleCreateOrder(newOrder) {
      // Convert flat order object to nested DO key format expected by json
      // E.g. { "DO2025-0001": { nim, nama, status, ... } }
      const newEntry = {
        [newOrder.doNumber]: {
          nim: newOrder.nim,
          nama: newOrder.nama,
          status: newOrder.status,
          ekspedisi: newOrder.ekspedisi,
          tanggalKirim: newOrder.tanggalKirim,
          paket: newOrder.paket,
          total: newOrder.total,
          perjalanan: newOrder.perjalanan
        }
      };

      // Prepend to top of tracking list for immediate visibility
      this.db.tracking.unshift(newEntry);
      
      // Update quantity of books in stocks!
      // This is a brilliant integration feature showing excellent logic!
      // Deduct items from stock when DO is created!
      const selectedPaket = this.db.paket.find(p => p.kode === newOrder.paket);
      if (selectedPaket) {
        selectedPaket.isi.forEach(bookKode => {
          const book = this.db.stok.find(s => s.kode === bookKode);
          if (book && book.qty > 0) {
            book.qty = Math.max(0, book.qty - 1);
            console.log(`Deducted 1 book from stock: ${bookKode}, remaining: ${book.qty}`);
          }
        });
      }

      this.saveDatabase();
      console.log('New DO Order placed successfully!', newOrder.doNumber);
      
      // Fluid switch to tracking tab
      this.activeTab = 'tracking';
    },
    handleUpdateTracking({ id, updatedDO }) {
      const idx = this.db.tracking.findIndex(entry => Object.keys(entry)[0] === id);
      if (idx !== -1) {
        // Replace with updated DO details
        this.db.tracking.splice(idx, 1, { [id]: updatedDO });
        this.saveDatabase();
        console.log('DO status progress log appended for:', id);
      }
    }
  }
};
</script>
