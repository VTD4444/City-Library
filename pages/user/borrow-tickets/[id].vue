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
                    <span>{{ ticket.TenDocGia }}</span>
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
              </v-card-title>

              <v-data-table
                :headers="tableHeaders"
                :items="borrowedBooks"
                class="borrowed-books-table"
                hide-default-footer
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
        TenDocGia: "",
        MaDocGia: "",
        NgayYeuCau: null,
        NgayPhaiTra: null,
        TrangThaiPhieu: "",
      },
      borrowedBooks: [],
      originalBooks: [],
      loading: true,
      error: null,
      saving: false,

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
    };
  },
  computed: {
    // Get the ticket ID from the route params
    routeTicketId() {
      return this.$route.params.id;
    },
  },
  async mounted() {
    this.ticketId = this.routeTicketId;
    await this.fetchTicketDetails();
  },
  methods: {
    async fetchTicketDetails() {
      this.loading = true;
      this.error = null;

      try {
        const response = await fetch(
          `http://26.193.242.15:8000/phieumuons/lay1phieumuon/${this.ticketId}`,
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
          TenDocGia: data.TenDocGia,
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
</style>
