<template>
  <div class="history-page">
    <v-container>
      <h1 class="text-h4 text-center font-weight-bold mb-6">
        LỊCH SỬ MƯỢN SÁCH
      </h1>

      <!-- Hiển thị khi không có phiếu mượn -->
      <v-card v-if="loading" elevation="0"class="pa-6 text-center">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
        <p class="mt-3">Đang tải dữ liệu...</p>
      </v-card>

      <v-card v-else-if="!loanHistory.length" class="pa-6 text-center">
        <v-icon size="64" color="grey" class="mb-4">mdi-book-off</v-icon>
        <h2 class="text-h5 mb-2">Bạn chưa có lịch sử mượn sách</h2>
        <p class="mb-6 text-body-1">
          Hãy mượn sách để xem lịch sử mượn sách tại đây
        </p>
        <v-btn color="primary" @click="$router.push('/books')">
          <v-icon start>mdi-book-open-variant</v-icon>
          Tìm sách để mượn
        </v-btn>
      </v-card>

      <!-- Hiển thị khi có phiếu mượn -->
      <template v-else>
        <!-- Danh sách phiếu mượn -->
        <v-card elevation="0" class="mb-6">
          <v-data-table
            :headers="tableHeaders"
            :items="loanHistoryWithIndex"
            class="history-table"
            :items-per-page="pageSize"
            hide-default-footer
          >
            <!-- Cột STT -->
            <template v-slot:item.stt="{ item }">
              {{ item.stt }}
            </template>

            <!-- Cột mã phiếu mượn -->
            <template v-slot:item.MaPhieuMuon="{ item }">
              {{ item.MaPhieuMuon }}
            </template>

            <!-- Cột ngày mượn -->
            <template v-slot:item.NgayYeuCau="{ item }">
              {{ formatDate(item.NgayYeuCau) }}
            </template>

            <!-- Cột hạn trả -->
            <template v-slot:item.NgayPhaiTra="{ item }">
              {{ formatDate(item.NgayPhaiTra) }}
            </template>

            <!-- Cột trạng thái -->
            <template v-slot:item.TrangThaiPhieu="{ item }">
              <v-chip
                :color="getStatusColor(item.TrangThaiPhieu)"
                size="small"
                text-color="white"
              >
                {{ getStatusText(item.TrangThaiPhieu) }}
              </v-chip>
            </template>

            <!-- Cột xem chi tiết -->
            <template v-slot:item.actions="{ item }">
              <v-btn
                variant="text"
                color="primary"
                size="small"
                @click="viewLoanDetail(item.MaPhieuMuon)"
              >
                Xem
              </v-btn>
            </template>
          </v-data-table>
        </v-card>

        <!-- Phân trang -->
        <div class="d-flex justify-center">
          <v-pagination
            v-model="currentPage"
            :length="totalPages"
            @update:modelValue="handlePageChange"
            rounded="circle"
          ></v-pagination>
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
  name: "HistoryPage",
  data() {
    return {
      loanHistory: [],
      loading: true,
      currentPage: 1,
      pageSize: 10,
      totalItems: 0,
      tableHeaders: [
        {
          title: "STT",
          key: "stt",
          sortable: false,
          align: "center",
          width: "80px",
        },
        {
          title: "Mã phiếu mượn",
          key: "MaPhieuMuon",
          sortable: false,
          align: "center",
          width: "150px",
        },
        {
          title: "Ngày mượn",
          key: "NgayYeuCau",
          align: "center",
          width: "150px",
        },
        {
          title: "Hạn trả",
          key: "NgayPhaiTra",
          align: "center",
          width: "150px",
        },
        {
          title: "Trạng thái",
          key: "TrangThaiPhieu",
          sortable: false,
          align: "center",
          width: "150px",
        },
        {
          title: "Phiếu mượn",
          key: "actions",
          sortable: false,
          align: "center",
          width: "100px",
        },
      ],
      filters: {
        trang_thai: "",
        ma_doc_gia: "",
      },
      loanDetailDialog: {
        show: false,
        data: null,
        books: [],
      },
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },
    };
  },
  computed: {
    // Thêm STT vào mỗi item để hiển thị
    loanHistoryWithIndex() {
      const startIndex = (this.currentPage - 1) * this.pageSize;
      return this.loanHistory.map((item, index) => ({
        ...item,
        stt: startIndex + index + 1,
      }));
    },
    totalPages() {
      return Math.ceil(this.totalItems / this.pageSize);
    },
  },
  mounted() {
    this.fetchLoanHistory();
  },
  methods: {
    async fetchLoanHistory() {
      const userLogin = JSON.parse(localStorage.getItem("userLogin") || "null");
      if (!userLogin) {
        this.$router.push("/login");
        return;
      }
      this.loading = true;

      try {
        const requestData = {
          IDBanDoc: userLogin.MaDocGia, // Sửa từ ma_doc_gia thành IDBanDoc theo API
          trang_thai: this.filters.trang_thai || undefined,
          page: this.currentPage,
          page_size: this.pageSize,
        };

        const response = await fetch(
          "https://26.193.242.15:8080/phieumuons/filter",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(requestData),
          }
        );

        const data = await response.json();
        console.log("Loan history response:", data);

        if (!response.ok) {
          throw new Error(data.detail || "Không thể tải lịch sử mượn sách");
        }

        // Kiểm tra xem data có phải là một mảng không
        if (Array.isArray(data)) {
          this.loanHistory = data;
          this.totalItems = data.length;
        } else {
          // Nếu API thay đổi định dạng trở lại kiểu cũ
          this.loanHistory = data.data || [];
          this.totalItems = data.total || this.loanHistory.length;
        }
      } catch (error) {
        console.error("Error fetching loan history:", error);
        this.snackbar.text =
          "Không thể tải lịch sử mượn sách: " + error.detail;
        this.snackbar.color = "error";
        this.snackbar.show = true;
        this.loanHistory = [];
      } finally {
        this.loading = false;
      }
    },

    handlePageChange() {
      this.fetchLoanHistory();
    },

    formatDate(dateString) {
      if (!dateString) return "";

      const date = new Date(dateString);
      return date.toLocaleDateString("vi-VN", {
        year: "numeric",
        month: "2-digit",
        day: "2-digit",
      });
    },

    getStatusColor(status) {
      switch (status) {
        case "ChoXacNhan":
          return "orange";
        case "DaXacNhan":
          return "blue";
        case "DangMuon":
          return "warning";
        case "DaTra":
          return "success";
        case "DaHuy":
          return "error";
        default:
          return "grey";
      }
    },

    getStatusText(status) {
      switch (status) {
        case "ChoXacNhan":
          return "Chờ xác nhận";
        case "DaXacNhan":
          return "Đã xác nhận";
        case "DangMuon":
          return "Đang mượn";
        case "DaTra":
          return "Đã trả";
        case "DaHuy":
          return "Đã hủy";
        default:
          return status;
      }
    },

    viewLoanDetail(loanId) {
      this.$router.push(`/user/borrow-tickets/${loanId}`);
    },
  },
};
</script>

<style scoped>
.history-page {
  padding-bottom: 40px;
}

.history-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e3f2fd !important;
}

.history-table :deep(td) {
  height: 60px;
}

/* Tạo border cho bảng */
.history-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.history-table :deep(th),
.history-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}
</style>
