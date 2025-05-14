<template>
  <div class="borrow-tickets-tab">
    <!-- Header Bar with Search -->
    <div class="d-flex flex-wrap align-center justify-space-between mb-4">
      <div class="search-container d-flex ml-auto">
        <v-text-field
          v-model="search"
          placeholder="Nhập mã phiếu mượn/tên độc giả"
          density="compact"
          variant="outlined"
          hide-details
          append-inner-icon="mdi-magnify"
          single-line
          class="search-field"
        ></v-text-field>
      </div>
    </div>

    <!-- Loading state -->
    <v-card v-if="loading" elevation="0" class="pa-6 text-center">
      <v-progress-circular indeterminate color="primary"></v-progress-circular>
      <div class="mt-2">Đang tải dữ liệu phiếu mượn...</div>
    </v-card>

    <!-- Error state -->
    <v-alert v-else-if="error" type="error" class="mb-6">
      {{ error }}
    </v-alert>

    <!-- Empty state -->
    <v-card
      v-else-if="!allBorrowTickets.length"
      elevation="0"
      class="pa-6 text-center"
    >
      <v-icon size="64" color="purple" class="mb-4">mdi-file-document</v-icon>
      <h2 class="text-h5 mb-2">Không có phiếu mượn</h2>
      <p class="mb-6 text-body-1">
        Hiện tại không có phiếu mượn nào trong hệ thống
      </p>
    </v-card>

    <!-- Borrow Tickets Table -->
    <v-card v-else elevation="0">
      <div class="table-responsive">
        <v-data-table
          :headers="tableHeaders"
          :items="paginatedBorrowTickets"
          class="borrow-tickets-table"
          hide-default-footer
        >
          <!-- STT Column -->
          <template v-slot:item.stt="{ item }">
            {{ item.stt }}
          </template>

          <!-- Borrow Ticket ID Column -->
          <template v-slot:item.MaPhieuMuon="{ item }">
            {{ item.MaPhieuMuon }}
          </template>

          <!-- Reader Name Column -->
          <template v-slot:item.BanDoc="{ item }">
            {{ item.HoTen }}
          </template>

          <!-- Email Column -->
          <template v-slot:item.Email="{ item }">
            {{ item.Email }}
          </template>

          <!-- Phone Number Column -->
          <template v-slot:item.SoDienThoai="{ item }">
            {{ item.SoDienThoai }}
          </template>

          <!-- Borrow Date Column -->
          <template v-slot:item.NgayMuon="{ item }">
            {{ formatDate(item.NgayYeuCau) }}
          </template>

          <!-- Expected Return Date Column -->
          <template v-slot:item.NgayTraDuKien="{ item }">
            {{ formatDate(item.NgayPhaiTra) }}
          </template>

          <!-- Book Count Column -->
          <template v-slot:item.SoLuongSachMuon="{ item }">
            {{ item.SoLuongSachMuon || 1 }}
          </template>

          <!-- Status Column -->
          <template v-slot:item.TrangThai="{ item }">
            <v-select
              v-model="item.TrangThaiPhieu"
              :items="statusOptions"
              density="compact"
              variant="outlined"
              hide-details
              class="status-select"
              :class="getStatusClass(item.TrangThaiPhieu)"
              @update:model-value="updateStatus(item)"
            ></v-select>
          </template>

          <!-- Actions Column -->
          <template v-slot:item.actions="{ item }">
            <div class="d-flex justify-center">
              <v-btn
                variant="text"
                color="green-darken-1"
                size="small"
                class="action-button"
                @click="viewTicketDetail(item.MaPhieuMuon)"
              >
                Xem
              </v-btn>
              <v-btn
                variant="text"
                color="red"
                size="small"
                class="action-button"
                @click="deleteTicket(item)"
              >
                Xóa
              </v-btn>
            </div>
          </template>
        </v-data-table>
      </div>

      <!-- Custom Pagination -->
      <div class="d-flex justify-center align-center pa-4">
        <v-btn
          color="green"
          elevation="0"
          icon
          @click="page--"
          :disabled="page === 1"
        >
          <v-icon>mdi-chevron-left</v-icon>
        </v-btn>
        <span class="mx-2"> Trang {{ page }} / {{ totalPages }} </span>
        <v-btn
          color="green"
          elevation="0"
          icon
          @click="page++"
          :disabled="page >= totalPages"
        >
          <v-icon>mdi-chevron-right</v-icon>
        </v-btn>
      </div>
    </v-card>

    <!-- Status Update Snackbar -->
    <v-snackbar v-model="snackbar.show" :color="snackbar.color" :timeout="3000">
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="snackbar.show = false">Đóng</v-btn>
      </template>
    </v-snackbar>

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
            :loading="deleteLoading"
          >
            Xóa
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  name: "BorrowTicketsTab",
  props: {
    searchQuery: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      // Table data
      allBorrowTickets: [],
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 10,
      search: "", // Local search state
      searchTimeout: null, // For debouncing search

      // Status options
      statusOptions: [
        { title: "Chờ xác nhận", value: "ChoXacNhan" },
        { title: "Đang mượn", value: "DangMuon" },
        { title: "Hoàn thành", value: "HoanThanh" },
        { title: "Quá hạn", value: "QuaHan" },
      ],

      // Table headers
      tableHeaders: [
        {
          title: "STT",
          key: "stt",
          align: "center",
          width: "50px",
          sortable: false,
        },
        {
          title: "Mã phiếu mượn",
          key: "MaPhieuMuon",
          align: "center",
          width: "130px",
        },
        { title: "Bạn đọc", key: "BanDoc", align: "left", width: "150px" },
        { title: "Email", key: "Email", align: "left", width: "200px" },
        {
          title: "Số điện thoại",
          key: "SoDienThoai",
          align: "center",
          width: "130px",
        },
        {
          title: "Ngày mượn",
          key: "NgayMuon",
          align: "center",
          width: "120px",
        },
        {
          title: "Ngày trả dự kiến",
          key: "NgayTraDuKien",
          align: "center",
          width: "150px",
        },
        {
          title: "SL sách mượn",
          key: "SoLuongSachMuon",
          align: "center",
          width: "80px",
        },
        {
          title: "Trạng thái",
          key: "TrangThai",
          align: "center",
          width: "120px",
        },
        {
          title: "Hành động",
          key: "actions",
          align: "center",
          width: "120px",
          sortable: false,
        },
      ],

      // Snackbar for notifications
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },

      // Delete dialog state
      deleteDialog: false,
      deleteLoading: false,
      deleteItem: null,
      ticketToDelete: null,
    };
  },
  computed: {
    // Filter borrow tickets based on search term
    filteredBorrowTickets() {
      if (!this.allBorrowTickets.length) return [];

      if (!this.search) {
        return this.allBorrowTickets;
      }

      const searchLower = this.search.toLowerCase().trim();
      return this.allBorrowTickets.filter(
        (ticket) =>
          (ticket.MaPhieuMuon &&
            ticket.MaPhieuMuon.toString().includes(searchLower)) ||
          (ticket.HoTen && ticket.HoTen.toLowerCase().includes(searchLower)) ||
          (ticket.Email && ticket.Email.toLowerCase().includes(searchLower))
      );
    },

    // Get the paginated borrow tickets
    paginatedBorrowTickets() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;

      return this.filteredBorrowTickets
        .slice(startIndex, endIndex)
        .map((ticket, index) => ({
          ...ticket,
          stt: startIndex + index + 1,
        }));
    },

    // Calculate total pages
    totalPages() {
      return Math.max(
        1,
        Math.ceil(this.filteredBorrowTickets.length / this.itemsPerPage)
      );
    },
  },
  watch: {
    // Reset to page 1 and fetch when search changes
    search(newVal) {
      this.page = 1;

      // Clear any existing timeout
      if (this.searchTimeout) {
        clearTimeout(this.searchTimeout);
      }

      // Set a timeout to avoid making API calls for every keystroke
      this.searchTimeout = setTimeout(() => {
        this.fetchBorrowTickets();
      }, 500); // 500ms delay
    },

    // Watch for changes in searchQuery from parent
    searchQuery: {
      handler(newVal) {
        this.search = newVal;
        this.page = 1;
      },
      immediate: true,
    },
  },
  mounted() {
    this.fetchBorrowTickets();
  },
  methods: {
    async fetchBorrowTickets() {
      this.loading = true;
      this.error = null;

      try {
        // Make API request to get borrow tickets
        const response = await fetch(
          "https://26.193.242.15:8080/phieumuons/filter",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Accept: "application/json",
            },
            body: JSON.stringify({
              TenDocGiaorMaPhieuMuon: this.search.trim(),
              trang_thai: "", // Empty to get all statuses
              page: 1,
              page_size: 1000, // Get a large number to fetch all at once
            }),
          }
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(
            errorData.message || "Không thể tải danh sách phiếu mượn"
          );
        }

        const data = await response.json();
        this.allBorrowTickets = Array.isArray(data) ? data : [];
      } catch (error) {
        console.error("Error fetching borrow tickets:", error);
        this.error = `Lỗi: ${
          error.message || "Không thể tải danh sách phiếu mượn"
        }`;
        this.allBorrowTickets = [];
      } finally {
        this.loading = false;
      }
    },

    formatDate(dateString) {
      if (!dateString) return "N/A";

      const date = new Date(dateString);
      return date.toLocaleDateString("vi-VN", {
        day: "2-digit",
        month: "2-digit",
        year: "numeric",
      });
    },

    getStatusClass(status) {
      switch (status) {
        case "ChoXacNhan":
          return "status-pending";
        case "DangMuon":
          return "status-borrowed";
        case "DaTra":
          return "status-returned";
        default:
          return "";
      }
    },

    async updateStatus(item) {
      try {
        // Make API request to update ticket status
        const response = await fetch(
          `https://26.193.242.15:8080/phieumuons/${item.MaPhieuMuon}`,
          {
            method: "PATCH",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              TrangThaiPhieu: item.TrangThaiPhieu,
            }),
          }
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể cập nhật trạng thái");
        }

        this.snackbar = {
          show: true,
          text: "Cập nhật trạng thái thành công!",
          color: "success",
        };
      } catch (error) {
        console.error("Error updating status:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể cập nhật trạng thái"}`,
          color: "error",
        };
      }
    },

    viewTicketDetail(ticketId) {
      // Navigate directly to ticket detail page using Vue Router
      this.$router.push(`/admin/borrow-ticket/${ticketId}`);
    },

    deleteTicket(ticket) {
      // Set the ticket to delete and show the confirmation dialog
      this.ticketToDelete = ticket;
      this.deleteDialog = true;
    },

    async confirmDelete() {
      if (!this.ticketToDelete) return;

      this.deleteLoading = true;
      try {
        // Send delete request to API
        const response = await fetch(
          `https://26.193.242.15:8080/phieumuons/xoa-phieu-muon/${this.ticketToDelete.MaPhieuMuon}`,
          {
            method: "DELETE",
            headers: {
              Accept: "application/json",
            },
          }
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(
            errorData.message || "Không thể xóa phiếu mượn"
          );
        }

        // Show success message
        this.snackbar = {
          show: true,
          text: "Phiếu mượn đã được xóa thành công",
          color: "success",
        };

        // Close the dialog and refresh the list
        this.deleteDialog = false;
        this.ticketToDelete = null;

        // Refresh the list of tickets
        await this.fetchBorrowTickets();
      } catch (error) {
        console.error("Error deleting ticket:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể xóa phiếu mượn"}`,
          color: "error",
        };
      } finally {
        this.deleteLoading = false;
      }
    },
  },
};
</script>

<style scoped>
.borrow-tickets-tab {
  padding-bottom: 40px;
}

/* Table styling similar to the other tabs */
.borrow-tickets-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

/* Styling for table */
.borrow-tickets-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.borrow-tickets-table :deep(th),
.borrow-tickets-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

/* Status select styling */
.status-select {
  min-width: 120px;
}

.status-pending :deep(.v-field) {
  background-color: rgba(255, 193, 7, 0.1) !important;
}

.status-borrowed :deep(.v-field) {
  background-color: rgba(76, 175, 80, 0.1) !important;
}

.status-returned :deep(.v-field) {
  background-color: rgba(66, 165, 245, 0.1) !important;
}

.action-button {
  min-width: 60px;
}

/* Search field styling */
.search-container {
  max-width: 500px;
}

.search-field {
  min-width: 300px;
}

@media (max-width: 600px) {
  .search-field {
    min-width: 200px;
  }
}

/* Table responsive container */
.table-responsive {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

/* Pagination styling */
:deep(.v-pagination__item--active) {
  background-color: #2e7d32 !important;
  color: white !important;
}

/* Add this to your existing styles */
.notification-header {
  background-color: #f8f8f8;
  color: rgba(0, 0, 0, 0.87);
  font-weight: 500;
  padding: 16px;
}
</style>
