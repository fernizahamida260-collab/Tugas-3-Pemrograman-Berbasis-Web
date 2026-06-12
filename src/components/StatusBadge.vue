<template>
  <div class="tooltip-container">
    <span :class="['badge-status', statusClass]">
      <i :class="statusIcon"></i>
      {{ statusText }}
    </span>
    <!-- Tooltip content showing notes safely using v-html -->
    <div class="tooltip-content" v-if="catatanHtml" v-html="catatanHtml"></div>
    <div class="tooltip-content" v-else><em>Tidak ada catatan tambahan</em></div>
  </div>
</template>

<script>
export default {
  name: 'StatusBadge',
  props: {
    qty: {
      type: Number,
      required: true
    },
    safety: {
      type: Number,
      required: true
    },
    catatanHtml: {
      type: String,
      default: ''
    }
  },
  computed: {
    statusType() {
      if (this.qty === 0) {
        return 'kosong';
      } else if (this.qty < this.safety) {
        return 'menipis';
      } else {
        return 'aman';
      }
    },
    statusClass() {
      return {
        'badge-aman': this.statusType === 'aman',
        'badge-menipis': this.statusType === 'menipis',
        'badge-kosong': this.statusType === 'kosong'
      };
    },
    statusText() {
      if (this.statusType === 'kosong') return 'Kosong';
      if (this.statusType === 'menipis') return 'Menipis';
      return 'Aman';
    },
    statusIcon() {
      if (this.statusType === 'kosong') return 'fas fa-exclamation-triangle';
      if (this.statusType === 'menipis') return 'fas fa-exclamation-circle';
      return 'fas fa-check-circle';
    }
  }
};
</script>
