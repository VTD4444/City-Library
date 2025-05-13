<template>
  <div class="book-loan-detail-page">
    <v-container>
      <h1 class="text-h4 text-center font-weight-bold mb-6">PHIẾU MƯỢN</h1>

      <!-- Loading state -->
      <v-card v-if="loading" elevation="0" class="pa-6 text-center">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
        <p class="mt-3">Đang tải dữ liệu...</p>
      </v-card>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Content when data is loaded -->
      <template v-else-if="loanDetail">
        <!-- Layout hai cột -->
        <div class="loan-content-container d-flex flex-wrap">
          <!-- Cột bên trái: Thông tin phiếu mượn -->
          <div class="loan-info-column">
            <div class="info-row mb-2">
              <span class="info-label">Mã phiếu mượn:</span>
              <span class="info-value">{{ formatId(loanDetail.MaPhieuMuon) }}</span>
            </div>
            
            <div class="info-row mb-2">
              <span class="info-label">Tên bạn đọc:</span>
              <span class="info-value">{{ loanDetail.chi_tiet_phieu_muons?.[0]?.TenBanDoc || 'N/A' }}</span>
            </div>
            
            <div class="info-row mb-2">
              <span class="info-label">Mã bạn đọc:</span>
              <span class="info-value">{{ loanDetail.MaDocGia }}</span>
            </div>
            
            <div class="info-row mb-2">
              <span class="info-label">Ngày mượn:</span>
              <span class="info-value">{{ formatDate(loanDetail.NgayYeuCau) }}</span>
            </div>
            
            <div class="info-row mb-2">
              <span class="info-label">Hạn trả:</span>
              <span class="info-value">{{ formatDate(loanDetail.NgayPhaiTra) }}</span>
            </div>
          </div>
          
          <!-- Cột bên phải: Danh sách sách mượn -->
          <div class="books-table-column">
            <v-table class="books-table">
              <thead>
                <tr>
                  <th class="text-center">STT</th>
                  <th class="text-center">Mã sách</th>
                  <th>Tên sách</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(book, index) in loanDetail.chi_tiet_phieu_muons" :key="index">
                  <td class="text-center">{{ index + 1 }}</td>
                  <td class="text-center">{{ book.MaSach }}</td>
                  <td>{{ book.TenSach }}</td>
                </tr>
              </tbody>
            </v-table>
            
            <div class="text-right mt-3">
              <p class="text-subtitle-1 font-weight-bold text-success">
                Tổng: {{ loanDetail.chi_tiet_phieu_muons?.length || 0 }}
              </p>
            </div>
          </div>
        </div>
      </template>
    </v-container>

    <!-- Snackbar thông báo -->
    <v-snackbar v-model="snackbar.show" :color="snackbar.color" :timeout="3000">
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="snackbar.show = false">Đóng</v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>
export default {
  name: "BookLoanDetail",
  data() {
    return {
      loanId: null,
      loanDetail: null,
      loading: true,
      error: null,
      snackbar: {
        show: false,
        text: "",
        color: "success"
      }
    };
  },
  mounted() {
    this.loanId = this.$route.params.id;
    this.fetchLoanDetail();
  },
  methods: {
    async fetchLoanDetail() {
      this.loading = true;
      this.error = null;
      
      try {
        const response = await fetch(`http://26.193.242.15:8000/phieumuons/lay1phieumuon/${this.loanId}`);
        
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể tải thông tin phiếu mượn");
        }
        
        const data = await response.json();
        this.loanDetail = data;
        
      } catch (error) {
        console.error("Error fetching loan details:", error);
        this.error = "Không thể tải thông tin phiếu mượn: " + error.message;
      } finally {
        this.loading = false;
      }
    },
    
    formatId(id) {
      // Format mã phiếu mượn thành dạng "M" + id + các số 0 phía trước để đủ 6 chữ số
      if (!id) return '';
      return 'M' + id.toString().padStart(6, '0');
    },
    
    formatDate(dateString) {
      if (!dateString) return 'Chờ xác nhận';
      
      const date = new Date(dateString);
      return date.toLocaleDateString('vi-VN', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric'
      });
    },
    
    formatTotalWithLeadingZero(number) {
      // Format số lượng tổng với số 0 phía trước để đủ 2 chữ số
      return number.toString().padStart(2, '0');
    }
  }
};
</script>

<style scoped>
.book-loan-detail-page {
  padding-bottom: 40px;
}

.loan-content-container {
  max-width: 1200px;
  margin: 0 auto;
  gap: 32px;
}

/* Cột thông tin bên trái */
.loan-info-column {
  flex: 0 0 300px;
}

/* Cột bảng bên phải */
.books-table-column {
  flex: 1;
}

/* Cách hiển thị trên màn hình nhỏ */
@media (max-width: 768px) {
  .loan-content-container {
    flex-direction: column;
  }
  
  .loan-info-column {
    margin-bottom: 20px;
    width: 100%;
  }
}

.info-row {
  display: flex;
  margin-bottom: 12px;
  line-height: 1.6;
}

.info-label {
  font-weight: 600;
  min-width: 140px;
  margin-right: 10px;
  color: rgba(0, 0, 0, 0.7);
}

.books-table {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.books-table th {
  background-color: #e8f5e9 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  font-weight: 600 !important;
  white-space: nowrap;
}

.books-table th, 
.books-table td {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

.text-success {
  color: #2e7d32 !important;
}
</style>