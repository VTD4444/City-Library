<template>
  <div class="all-readers-tab">
    <v-container>
        <h1 class="text-h4 text-center font-weight-bold mb-6">QUẢN LÝ BẠN ĐỌC</h1>
      <!-- Header Bar with Search -->
      <div class="d-flex flex-wrap align-center justify-space-between mb-4">
        <div class="search-container d-flex ml-auto">
          <v-text-field
            v-model="search"
            placeholder="Tìm kiếm theo tên/email"
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
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
        <div class="mt-2">Đang tải danh sách bạn đọc...</div>
      </v-card>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Empty state -->
      <v-card
        v-else-if="!filteredReaders.length"
        elevation="0"
        class="pa-6 text-center"
      >
        <v-icon size="64" color="grey" class="mb-4">mdi-account-off</v-icon>
        <h2 class="text-h5 mb-2">Không tìm thấy bạn đọc nào</h2>
        <p class="mb-6 text-body-1">
          Không có bạn đọc nào phù hợp với tiêu chí tìm kiếm
        </p>
      </v-card>

      <!-- Readers Table -->
      <v-card v-else elevation="0" max-width="900px" class="mx-auto">
        <div class="table-responsive">
          <v-data-table
            :headers="tableHeaders"
            :items="paginatedReaders"
            class="readers-table"
            hide-default-footer
          >
            <!-- STT Column -->
            <template v-slot:item.stt="{ item }">
              {{ item.stt }}
            </template>

            <!-- Name Column -->
            <template v-slot:item.HoTen="{ item }">
              {{ item.HoTen }}
            </template>

            <!-- Email Column -->
            <template v-slot:item.Email="{ item }">
              {{ item.Email }}
            </template>

            <!-- Registration Date Column -->
            <template v-slot:item.NgayDangKy="{ item }">
              {{ formatDate(item.NgayDangKy) }}
            </template>

            <!-- Actions Column -->
            <template v-slot:item.actions="{ item }">
              <div class="d-flex justify-center">
                <v-btn
                  variant="text"
                  color="green-darken-1"
                  size="small"
                  class="action-button"
                  @click="viewReaderDetail(item.MaDocGia)"
                >
                  Xem
                </v-btn>
                <v-btn
                  variant="text"
                  color="red"
                  size="small"
                  class="action-button"
                  @click="deleteReader(item)"
                >
                  Xóa
                </v-btn>
              </div>
            </template>
          </v-data-table>
        </div>

        <!-- Simplified Pagination -->
        <div class="d-flex justify-center align-center pa-4">
          <v-btn color="green" icon @click="page--" :disabled="page === 1">
            <v-icon>mdi-chevron-left</v-icon>
          </v-btn>

          <span class="mx-2"> Trang {{ page }} / {{ totalPages }} </span>

          <v-btn
            color="green"
            icon
            @click="page++"
            :disabled="page >= totalPages"
          >
            <v-icon>mdi-chevron-right</v-icon>
          </v-btn>
        </div>
      </v-card>

      <!-- Delete confirmation dialog -->
      <v-dialog v-model="deleteDialog.show" max-width="500px" persistent>
        <v-card>
          <v-card-title class="notification-header d-flex align-center">
            <span>Xác nhận xóa</span>
            <v-spacer></v-spacer>
            <v-btn elevation="0" icon @click="deleteDialog.show = false">
              <v-icon color="red">mdi-close</v-icon>
            </v-btn>
          </v-card-title>

          <v-divider></v-divider>

          <v-card-text class="pt-4 pb-4">
            <p class="text-body-1">
              Bạn có chắc chắn muốn xóa bạn đọc
              <strong>{{ deleteDialog.reader?.HoTen }}</strong
              >?
            </p>
            <p class="text-body-2 mt-2 text-red">
              Hành động này không thể hoàn tác.
            </p>
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions class="justify-center pa-4">
            <v-btn
              color="green"
              min-width="80"
              variant="flat"
              @click="confirmDelete"
              :loading="deleteDialog.loading"
            >
              Xác nhận
            </v-btn>
            <v-btn
              color="grey"
              variant="outlined"
              min-width="80"
              class="ml-4"
              @click="deleteDialog.show = false"
            >
              Hủy
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- Snackbar for notifications -->
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
  name: "AllReadersTab",
  props: {
    searchQuery: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      // Table data
      allReaders: [],
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 10,
      search: "", // Local search state
      searchTimeout: null,

      // Table headers
      tableHeaders: [
        {
          title: "STT",
          key: "stt",
          align: "center",
          width: "60px",
          sortable: false,
        },
        { title: "Tên bạn đọc", key: "HoTen", align: "left", width: "150px" },
        { title: "Email", key: "Email", align: "left", width: "150px" },
        {
          title: "Ngày tham gia",
          key: "NgayDangKy",
          align: "center",
          width: "150px",
        },
        {
          title: "Hành động",
          key: "actions",
          align: "center",
          width: "150px",
          sortable: false,
        },
      ],

      // Delete dialog
      deleteDialog: {
        show: false,
        reader: null,
        loading: false,
      },

      // Snackbar
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },
    };
  },
  computed: {
    // Filter readers based on search term
    filteredReaders() {
      if (!this.search) {
        return this.allReaders;
      }

      const searchLower = this.search.toLowerCase().trim();
      return this.allReaders.filter(
        (reader) =>
          (reader.HoTen || "").toLowerCase().includes(searchLower) ||
          (reader.Email || "").toLowerCase().includes(searchLower) ||
          (reader.TenDangNhap || "").toLowerCase().includes(searchLower) ||
          (reader.SoDienThoai || "").includes(searchLower)
      );
    },

    // Get the paginated readers
    paginatedReaders() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;

      return this.filteredReaders
        .slice(startIndex, endIndex)
        .map((reader, index) => ({
          ...reader,
          stt: startIndex + index + 1,
        }));
    },

    // Calculate total pages
    totalPages() {
      return Math.max(
        1,
        Math.ceil(this.filteredReaders.length / this.itemsPerPage)
      );
    },
  },
  watch: {
    // Watch for changes in parent's searchQuery to update local search
    searchQuery: {
      handler(newVal) {
        this.search = newVal;
      },
      immediate: true,
    },

    // Reset to page 1 when search changes
    search() {
      this.page = 1;
    },
  },
  mounted() {
    this.fetchAllReaders();
  },
  methods: {
    async fetchAllReaders() {
      this.loading = true;
      this.error = null;

      try {
        // Make API request to get all readers
        const response = await fetch(
          "https://26.193.242.15:8080/bandoc/get_all"
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(
            errorData.message || "Không thể tải danh sách bạn đọc"
          );
        }

        const data = await response.json();
        this.allReaders = Array.isArray(data) ? data : [];
      } catch (error) {
        console.error("Error fetching readers:", error);
        this.error = `Lỗi: ${
          error.message || "Không thể tải danh sách bạn đọc"
        }`;
        this.allReaders = [];
      } finally {
        this.loading = false;
      }
    },

    formatDate(dateString) {
      if (!dateString) return "N/A";

      try {
        const date = new Date(dateString);
        return date.toLocaleDateString("vi-VN", {
          day: "2-digit",
          month: "2-digit",
          year: "numeric",
        });
      } catch (error) {
        return dateString;
      }
    },

    viewReaderDetail(readerId) {
      this.$router.push(`/admin/readers/${readerId}`);
    },

    deleteReader(reader) {
      this.deleteDialog.reader = reader;
      this.deleteDialog.show = true;
    },

    async confirmDelete() {
      if (!this.deleteDialog.reader) return;

      this.deleteDialog.loading = true;

      try {
        // Call the API to delete the reader
        const response = await fetch(`https://26.193.242.15:8080/bandoc/delete_user?user_id=${this.deleteDialog.reader.MaDocGia}`, {
          method: 'DELETE',
          headers: {
            'Accept': 'application/json'
          }
        });
        
        const data = await response.json();
        
        // Handle potential error responses
        if (!response.ok) {
          let errorMsg = "Không thể xóa bạn đọc";
          if (response.status === 404) {
            errorMsg = data.detail || "Không tìm thấy độc giả";
          } else if (data.detail) {
            errorMsg = data.detail;
          }
          throw new Error(errorMsg);
        }

        // Remove the reader from the local list
        this.allReaders = this.allReaders.filter(
          (r) => r.MaDocGia !== this.deleteDialog.reader.MaDocGia
        );

        // Show success message
        this.snackbar = {
          show: true,
          text: `Đã xóa bạn đọc "${this.deleteDialog.reader.HoTen}" thành công`,
          color: "success",
        };

        // Close dialog and reset
        this.deleteDialog.show = false;
        this.deleteDialog.reader = null;
      } catch (error) {
        console.error("Error deleting reader:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể xóa bạn đọc"}`,
          color: "error",
        };
        this.deleteDialog.show = false;
      } finally {
        this.deleteDialog.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.all-readers-tab {
  padding-bottom: 40px;
}

.search-container {
  max-width: 400px;
}

.search-field {
  min-width: 250px;
}

@media (max-width: 600px) {
  .search-field {
    min-width: 180px;
  }
}

.readers-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

/* Styling for table */
.readers-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.readers-table :deep(th),
.readers-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

/* Animation for table rows */
.readers-table :deep(tbody tr) {
  transition: background-color 0.2s ease;
}

.readers-table :deep(tbody tr:hover) {
  background-color: rgba(76, 175, 80, 0.05);
}

/* Table responsive container */
.table-responsive {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

/* Set minimum width for the table */
.readers-table {
  min-width: 800px;
}

.notification-header {
  background-color: #f8f8f8;
  color: rgba(0, 0, 0, 0.87);
  font-weight: 500;
  padding: 16px;
}

.action-button {
  min-width: 60px;
}
</style>
