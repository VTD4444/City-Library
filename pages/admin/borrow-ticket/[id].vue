<template>
  <div class="borrow-ticket-page">
    <v-container>
      <!-- Back button and page title -->
      <div class="d-flex align-center mb-6">
        <v-btn
          prepend-icon="mdi-arrow-left"
          variant="text"
          @click="$router.back()"
          class="mr-4"
        >
          Quay lại
        </v-btn>
        <h1 class="text-h4 text-center flex-grow-1 mr-16">PHIẾU MƯỢN</h1>
      </div>

      <!-- Loading state -->
      <v-card v-if="loading" elevation="0" class="pa-6 text-center">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
        <div class="mt-2">Đang tải thông tin phiếu mượn...</div>
      </v-card>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Borrow ticket details - Side by side layout -->
      <template v-else>
        <v-row>
          <!-- Left column: Ticket information -->
          <v-col cols="12" md="4" lg="4">
            <v-card elevation="0" class="h-100">
              <v-card-title class="pb-5">Thông tin phiếu mượn</v-card-title>
              <v-card-text>
                <div class="ticket-details">
                  <div class="ticket-info-row mb-3">
                    <strong class="info-label">Mã phiếu mượn:</strong>
                    <span>{{ ticket.MaPhieuMuon }}</span>
                  </div>

                  <div class="ticket-info-row mb-3">
                    <strong class="info-label">Tên bạn đọc:</strong>
                    <span>{{ ticket.TenBanDoc }}</span>
                  </div>

                  <div class="ticket-info-row mb-3">
                    <strong class="info-label">Mã thẻ thư viện:</strong>
                    <span>{{ ticket.MaDocGia || "Chưa có thông tin" }}</span>
                  </div>

                  <div class="ticket-info-row mb-3">
                    <strong class="info-label">Ngày mượn:</strong>
                    <span>{{ formatDate(ticket.NgayYeuCau) }}</span>
                  </div>

                  <div class="ticket-info-row mb-3">
                    <strong class="info-label">Hạn trả:</strong>
                    <span>{{ formatDate(ticket.NgayPhaiTra) }}</span>
                  </div>
                </div>
              </v-card-text>
            </v-card>
          </v-col>

          <!-- Right column: Books table -->
          <v-col cols="12" md="8" lg="8">
            <v-card elevation="0">
              <v-card-title class="d-flex justify-space-between align-center pb-5">
                Danh sách sách mượn

                <!-- Update the Return Button to check ticket status -->
                <v-btn
                  color="green-darken-1"
                  :disabled="selectedItems.length === 0 || ticket.TrangThaiPhieu === 'ChoXacNhan'"
                  @click="paid"
                  prepend-icon="mdi-check-circle"
                >
                  Đã trả
                </v-btn>
              </v-card-title>

              <v-data-table
                v-model="selectedItems"
                :headers="tableHeaders"
                :items="borrowedBooks"
                class="borrowed-books-table"
                show-select
                hide-default-footer
                item-value="MaSach"
              >
                <!-- STT Column -->
                <template v-slot:item.stt="{ index }">
                  {{ index + 1 }}
                </template>

                <!-- Book ID Column -->
                <template v-slot:item.MaSach="{ item }">
                  {{ item.MaSach }}
                </template>

                <!-- Book Title Column -->
                <template v-slot:item.TenSach="{ item }">
                  {{ item.TenSach }}
                </template>

                <!-- Status Column -->
                <template v-slot:item.TrangThaiSach="{ item }">
                  <v-chip
                    size="small"
                    :color="getStatusChipColor(item.TrangThaiSach)"
                    :text-color="getStatusChipTextColor(item.TrangThaiSach)"
                  >
                    {{ formatStatus(item.TrangThaiSach) }}
                  </v-chip>
                </template>
              </v-data-table>

              <!-- Total books count -->
              <div class="pa-4 d-flex justify-end">
                <strong class="text-h6 green--text"
                  >Tổng: {{ borrowedBooks.length }}</strong
                >
              </div>
            </v-card>
          </v-col>
        </v-row>
      </template>

      <div class="d-flex justify-center mt-4">
        <v-btn 
          color="primary" 
          @click="confirmTicket" 
          :loading="loading"
          :disabled="ticket.TrangThaiPhieu !== 'ChoXacNhan'"
        >
          {{ ticket.TrangThaiPhieu === 'DaXacNhan' ? 'Đã xác nhận' : 'Xác nhận' }}
        </v-btn>
        <v-btn 
          color="error" 
          class="ml-4" 
          @click="deleteTicket"
          :loading="loading"
        >
          Xóa
        </v-btn>
      </div>

      <!-- Custom Delete Confirmation Dialog -->
      <v-dialog v-model="deleteDialog" max-width="500px" persistent>
        <v-card>
          <v-card-title class="notification-header d-flex align-center">
            <span>Thông báo:</span>
            <v-spacer></v-spacer>
            <v-btn elevation="0" icon @click="deleteDialog = false">
              <v-icon color="red">mdi-close</v-icon>
            </v-btn>
          </v-card-title>
          
          <v-divider></v-divider>
          
          <v-card-text class="pt-4 pb-4">
            <p class="text-body-1">Bạn có chắc chắn muốn xóa phiếu mượn này?</p>
            <p class="text-body-2 mt-2">Hành động này không thể hoàn tác.</p>
          </v-card-text>
          
          <v-divider></v-divider>
          
          <v-card-actions class="justify-center pa-4">
            <v-btn
              color="success"
              min-width="80"
              @click="confirmDelete"
              :loading="loading"
            >
              Xóa
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- Notification -->
      <v-snackbar
        v-model="snackbar.show"
        :color="snackbar.color"
        :timeout="3000"
      >
        {{ snackbar.text }}
        <template v-slot:actions>
          <v-btn variant="text" @click="snackbar.show = false">Đóng</v-btn>
        </template>
      </v-snackbar>
    </v-container>
  </div>
</template>

<script>
export default {
  name: "BorrowTicketDetail",
  data() {
    return {
      ticketId: null,
      ticket: {
        MaPhieuMuon: "",
        TenBanDoc: "",
        MaDocGia: "",
        NgayYeuCau: null,
        NgayPhaiTra: null,
        TrangThaiPhieu: "",
      },
      borrowedBooks: [],
      originalBooks: [],
      loading: true,
      error: null,

      // Table headers
      tableHeaders: [
        {
          title: "STT",
          key: "stt",
          align: "center",
          width: "50px",
          sortable: false,
        },
        { title: "Mã sách", key: "MaSach", align: "center", width: "120px" },
        { title: "Tên sách", key: "TenSach", align: "left" },
        { title: "Trạng thái", key: "TrangThaiSach", align: "center", width: "120px" }
      ],

      // Add book dialog
      showAddBookDialog: false,
      selectedBook: null,
      newBookQuantity: 1,
      availableBooks: [],

      // Notification
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },

      // Custom delete dialog
      deleteDialog: false,

      // Selected items for return
      selectedItems: [],
      returnDialog: {
        loading: false
      },
    };
  },
  computed: {
    // Get the ticket ID from the route params
    routeTicketId() {
      return this.$route.params.id;
    },

    // Selected books for return
    selectedBooks() {
      return this.borrowedBooks.filter((book) => 
        this.selectedItems.includes(book.MaChiTiet)
      );
    }
  },
  async mounted() {
    this.ticketId = this.routeTicketId;
    await this.fetchTicketDetails();
    await this.fetchAvailableBooks();
  },
  methods: {
    async fetchTicketDetails() {
      this.loading = true;
      this.error = null;

      try {
        const response = await fetch(
          `https://26.193.242.15:8080/phieumuons/lay1phieumuon/${this.ticketId}`,
          {
            method: "GET",
            headers: {
              Accept: "application/json",
            },
          }
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(
            errorData.message || "Không thể tải thông tin phiếu mượn"
          );
        }

        const data = await response.json();

        // Set ticket data
        this.ticket = {
          MaPhieuMuon: data.MaPhieuMuon,
          TenBanDoc: data.TenBanDoc,
          MaDocGia: data.MaDocGia,
          NgayYeuCau: data.NgayYeuCau,
          NgayPhaiTra: data.NgayPhaiTra,
          TrangThaiPhieu: data.TrangThaiPhieu,
        };

        // Set borrowed books
        if (
          data.chi_tiet_phieu_muons &&
          Array.isArray(data.chi_tiet_phieu_muons)
        ) {
          this.borrowedBooks = data.chi_tiet_phieu_muons.map((book) => ({
            MaChiTiet: book.MaChiTiet,
            MaSach: book.MaSach,
            TenSach: book.TenSach,
            TrangThaiSach: book.TrangThaiSach,
            SoLuong: book.SoLuong || 1,
          }));

          // Store original data for cancel functionality
          this.originalBooks = JSON.parse(JSON.stringify(this.borrowedBooks));
        }
      } catch (error) {
        console.error("Error fetching ticket details:", error);
        this.error = `Lỗi: ${
          error.message || "Không thể tải thông tin phiếu mượn"
        }`;
      } finally {
        this.loading = false;
      }
    },

    formatDate(dateString) {
      if (!dateString) return "Chưa xác định";

      const date = new Date(dateString);
      return date.toLocaleDateString("vi-VN", {
        day: "2-digit",
        month: "2-digit",
        year: "numeric",
      });
    },

    formatStatus(status) {
      switch (status) {
        case 'ChuaMuon':
          return 'Chưa mượn';
        case 'DangMuon':
          return 'Đang mượn';
        case 'DaTra':
          return 'Đã trả';
        case 'QuaHan':
          return 'Quá hạn';
      }
    },
    
    getStatusChipColor(status) {
      switch (status) {
        case 'ChuaMuon':
          return 'yellow-darken-4';
        case 'DangMuon':
          return 'green';
        case 'DaTra':
          return 'blue';
        case 'QuaHan':
          return 'red-lighten-5';
        default:
          return 'blue-lighten-5';
      }
    },

    getStatusChipTextColor(status) {
      switch (status) {
        case 'ChuaMuon':
          return 'amber-darken-4';
        case 'DangMuon':
          return 'white';
        case 'DaTra':
          return 'white';
        case 'QuaHan':
          return 'red-darken-4';
        default:
          return 'grey';
      }
    },

    async confirmTicket() {
      try {
        this.loading = true;
        
        // Get admin info from localStorage
        const adminData = JSON.parse(localStorage.getItem('adminLogin')) || {};
        const staffId = adminData.MaNV || 1; // Use MaNV from localStorage, fallback to 1 if not found
        
        // Prepare request body for API with the correct staff ID
        const requestBody = {
          "phieu_id": this.ticketId,
          "manv": staffId  // Use the actual staff ID from localStorage
        };
        
        // Send confirm request to API
        const response = await fetch('https://26.193.242.15:8080/phieumuons/xacnhan', {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
          },
          body: JSON.stringify(requestBody)
        });
        
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || 'Không thể xác nhận phiếu mượn');
        }
        
        // Show success message
        this.snackbar = {
          show: true,
          text: 'Phiếu mượn đã được xác nhận thành công',
          color: 'success'
        };
        
        // Refresh data
        await this.fetchTicketDetails();
        
      } catch (error) {
        console.error('Error confirming ticket:', error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || 'Không thể xác nhận phiếu mượn'}`,
          color: 'error'
        };
      } finally {
        this.loading = false;
      }
    },

    async deleteTicket() {
      // Show the custom dialog instead of using confirm()
      this.deleteDialog = true;
    },

    async confirmDelete() {
      try {
        this.loading = true;
        
        // Send delete request to API
        const response = await fetch(`https://26.193.242.15:8080/phieumuons/xoa-phieu-muon/${this.ticketId}`, {
          method: 'DELETE',
          headers: {
            'Accept': 'application/json'
          }
        });
        
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || 'Không thể xóa phiếu mượn');
        }
        
        // Show success message
        this.snackbar = {
          show: true,
          text: 'Phiếu mượn đã được xóa thành công',
          color: 'success'
        };
        
        // Close the dialog
        this.deleteDialog = false;
        
        // Navigate back to the tickets list after a short delay
        setTimeout(() => {
          this.$router.push('/admin/manage-books');
        }, 1500);
        
      } catch (error) {
        console.error('Error deleting ticket:', error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || 'Không thể xóa phiếu mượn'}`,
          color: 'error'
        };
        
        // Close the dialog
        this.deleteDialog = false;
      } finally {
        this.loading = false;
      }
    },
    
    async paid() {
      this.returnDialog.loading = true;
      
      try {
        console.log("Selected items for return:", this.selectedBooks);
        // Prepare request body with the format shown in the screenshot
        const requestBody = {
          MaPhieuMuon: this.ticketId,
          danhsachcapnhap: this.selectedBooks.map(book => ({
            MaChiTiet: book.MaSach,
            TrangThaiSachMuon: "DaTra"
          }))
        };
        
        // Call API to update book status
        const response = await fetch(
          "https://26.193.242.15:8080/phieumuons/chitietphieu/capnhat-trangthai",
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
              "Accept": "application/json"
            },
            body: JSON.stringify(requestBody)
          }
        );
        
        // Check response and handle errors
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.detail || errorData.message || "Không thể cập nhật trạng thái sách");
        }
        
        // Success handling
        this.snackbar = {
          show: true,
          text: `Đã cập nhật trạng thái ${this.selectedItems.length} quyển sách thành công`,
          color: "success"
        };
        
        // Update local data to reflect changes
        this.selectedItems.forEach(maChiTiet => {
          const book = this.borrowedBooks.find(b => b.MaChiTiet === maChiTiet);
          if (book) {
            book.TrangThaiSach = "DaTra";
          }
        });
        
        // Clear selection
        this.selectedItems = [];
        
        // Close dialog
        this.returnDialog.show = false;
        
        // Optionally refresh data from server
        // await this.fetchTicketDetails();
        
      } catch (error) {
        console.error("Error updating book status:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể cập nhật trạng thái sách"}`,
          color: "error"
        };
      } finally {
        this.returnDialog.loading = false;
      }
    }
  },
};
</script>

<style scoped>
.borrow-ticket-page {
  padding-bottom: 40px;
}

.ticket-details {
  max-width: 600px;
  margin: 0 auto;
}

.ticket-info-row {
  display: flex;
  align-items: flex-start;
}

.info-label {
  display: inline-block;
  width: 160px;
  font-weight: 500;
}

/* Table styling */
.borrowed-books-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

.borrowed-books-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.borrowed-books-table :deep(th),
.borrowed-books-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

.quantity-input {
  max-width: 80px;
  margin: 0 auto;
}

/* Custom delete dialog styles */
.notification-header {
  background-color: #f8f8f8;
  color: rgba(0, 0, 0, 0.87);
  font-weight: 500;
  padding: 16px;
}

/* Add styles for status chips */
.v-data-table :deep(.v-chip--size-small) {
  font-size: 12px;
}
</style>
