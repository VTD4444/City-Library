<template>
  <div class="readers-page">
    <v-container>
      <h1 class="text-h4 text-center font-weight-bold mb-6">QUẢN LÝ NGƯỜI DÙNG</h1>

      <!-- Header and Actions Row -->
      <div class="d-flex flex-wrap align-center justify-space-between mb-4">
        <!-- Right side: Search Bar -->
        <div class="search-container d-flex ml-auto">
          <v-text-field
            v-model="search"
            placeholder="Tìm kiếm người dùng"
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
        <div class="mt-2">Đang tải thông tin người dùng...</div>
      </v-card>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Empty state -->
      <v-card v-else-if="!allReaders.length" elevation="0" class="pa-6 text-center">
        <v-icon size="64" color="grey" class="mb-4">mdi-account-group</v-icon>
        <h2 class="text-h5 mb-2">Không có người dùng nào</h2>
        <p class="mb-6 text-body-1">
          Hiện tại chưa có người dùng nào trong hệ thống
        </p>
      </v-card>

      <!-- Readers Table -->
      <v-card v-else elevation="0" class="mx-auto">
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

            <!-- Join Date Column -->
            <template v-slot:item.NgayDangKy="{ item }">
              {{ formatDate(item.NgayDangKy) }}
            </template>

            <!-- Status Column -->
            <template v-slot:item.TrangThai="{ item }">
              <v-chip
                :color="getStatusColor(item.TrangThai)"
                :text-color="getStatusTextColor(item.TrangThai)"
                size="small"
                class="font-weight-medium"
              >
                {{ getStatusText(item.TrangThai) }}
              </v-chip>
            </template>

            <!-- Actions Column -->
            <template v-slot:item.actions="{ item }">
              <div class="d-flex gap-2">
                <v-btn
                  color="green"
                  variant="flat"
                  size="small"
                  @click="viewReader(item)"
                  class="px-3"
                >
                  Xem
                </v-btn>
                <v-btn
                  color="red"
                  variant="flat"
                  size="small"
                  @click="confirmDelete(item)"
                  class="px-3"
                >
                  Xóa
                </v-btn>
              </div>
            </template>
          </v-data-table>
        </div>

        <!-- Custom Pagination -->
        <div class="d-flex justify-center align-center pa-4">
          <v-btn color="green" elevation="0" icon @click="page--" :disabled="page === 1">
            <v-icon>mdi-chevron-left</v-icon>
          </v-btn>
          <span class="mx-2">
            Trang {{ page }} / {{ totalPages }}
          </span>
          <v-btn color="green" elevation="0" icon @click="page++" :disabled="page >= totalPages">
            <v-icon>mdi-chevron-right</v-icon>
          </v-btn>
        </div>
      </v-card>
    </v-container>

    <!-- Delete Confirmation Dialog -->
    <v-dialog v-model="deleteDialog.show" max-width="500">
      <v-card>
        <v-card-title class="text-h5 bg-grey-lighten-3 py-4 px-6">
          Xác nhận xóa tài khoản
        </v-card-title>
        <v-card-text class="pt-5">
          <p>Bạn có chắc chắn muốn xóa tài khoản của người dùng <strong>{{ deleteDialog.reader?.HoTen }}</strong>?</p>
          <p class="text-caption text-red mt-3">Lưu ý: Hành động này không thể hoàn tác.</p>
        </v-card-text>
        <v-card-actions class="pa-6 pt-0">
          <v-spacer></v-spacer>
          <v-btn
            color="grey"
            variant="text"
            @click="deleteDialog.show = false"
          >
            Hủy
          </v-btn>
          <v-btn
            color="red"
            variant="flat"
            @click="deleteReader"
            :loading="deleteDialog.loading"
          >
            Xác nhận xóa
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Snackbar thông báo -->
    <v-snackbar v-model="snackbar.show" :color="snackbar.color" :timeout="3000">
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="snackbar.show = false"> Đóng </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>
export default {
  name: "ReadersPage",
  data() {
    return {
      // Table data
      allReaders: [], // Store all readers
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 10,
      search: "",

      // Table headers
      tableHeaders: [
        { title: "STT", key: "stt", align: "center", width: "60px", sortable: false },
        { title: "Tên bạn đọc", key: "HoTen", align: "left" },
        { title: "Email", key: "Email", align: "left" },
        { title: "Ngày tham gia", key: "NgayDangKy", align: "center", width: "150px" },
        { title: "Trạng thái", key: "TrangThai", align: "center", width: "120px" },
        { title: "Hành động", key: "actions", align: "center", width: "150px", sortable: false },
      ],

      // Delete dialog
      deleteDialog: {
        show: false,
        reader: null,
        loading: false
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
      return this.allReaders.filter(reader => 
        reader.HoTen?.toLowerCase().includes(searchLower) ||
        reader.Email?.toLowerCase().includes(searchLower) ||
        reader.TenDangNhap?.toLowerCase().includes(searchLower)
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
          // Add a default status if not present
          TrangThai: this.determineStatus(reader)
        }));
    },
    
    // Calculate total pages
    totalPages() {
      return Math.ceil(this.filteredReaders.length / this.itemsPerPage);
    }
  },
  mounted() {
    this.fetchAllReaders();
  },
  watch: {
    // Reset to page 1 when search changes
    search() {
      this.page = 1;
    }
  },
  methods: {
    async fetchAllReaders() {
      this.loading = true;
      this.error = null;

      try {
        // Make API request to get all readers
        const response = await fetch("https://26.193.242.15:8080/bandoc/get_all");

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể tải danh sách người dùng");
        }

        const data = await response.json();
        this.allReaders = data;
      } catch (error) {
        console.error("Error fetching readers:", error);
        this.error = `Lỗi: ${error.message || "Không thể tải danh sách người dùng"}`;
        this.allReaders = [];
      } finally {
        this.loading = false;
      }
    },

    formatDate(dateString) {
      if (!dateString) return "N/A";
      
      try {
        const date = new Date(dateString);
        return new Intl.DateTimeFormat('vi-VN', {
          day: '2-digit',
          month: '2-digit',
          year: 'numeric'
        }).format(date);
      } catch (error) {
        return dateString;
      }
    },

    determineStatus(reader) {
      // Determine the reader's status - this could be replaced with actual status logic
      // For now, we'll assume all are active unless otherwise specified
      return reader.TrangThai || 'active';
    },

    getStatusText(status) {
      switch (status) {
        case 'active':
          return 'Đã cấp thẻ';
        case 'pending':
          return 'Chờ cấp thẻ';
        case 'inactive':
          return 'Chưa cấp thẻ';
        default:
          return 'Chưa cấp thẻ';
      }
    },

    getStatusColor(status) {
      switch (status) {
        case 'active':
          return 'green-lighten-5';
        case 'pending':
          return 'orange-lighten-5';
        case 'inactive':
          return 'red-lighten-5';
        default:
          return 'grey-lighten-3';
      }
    },

    getStatusTextColor(status) {
      switch (status) {
        case 'active':
          return 'green-darken-1';
        case 'pending':
          return 'orange-darken-1';
        case 'inactive':
          return 'red-darken-1';
        default:
          return 'grey-darken-1';
      }
    },

    viewReader(reader) {
      // Navigate to reader detail page
      this.$router.push(`/admin/readers/${reader.MaDocGia}`);
    },

    confirmDelete(reader) {
      this.deleteDialog.reader = reader;
      this.deleteDialog.show = true;
    },

    async deleteReader() {
      if (!this.deleteDialog.reader) return;
      
      this.deleteDialog.loading = true;
      
      try {
        // Here you would make an API call to delete the reader
        // For example:
        // const response = await fetch(`https://26.193.242.15:8080/bandoc/delete/${this.deleteDialog.reader.MaDocGia}`, {
        //   method: 'DELETE'
        // });
        
        // if (!response.ok) {
        //   throw new Error('Failed to delete reader');
        // }
        
        // Simulate API call for demonstration
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        // Remove reader from the list (in real app, you'd refetch or remove from array)
        this.allReaders = this.allReaders.filter(
          reader => reader.MaDocGia !== this.deleteDialog.reader.MaDocGia
        );
        
        // Show success message
        this.snackbar.color = 'success';
        this.snackbar.text = `Đã xóa tài khoản của ${this.deleteDialog.reader.HoTen}`;
        this.snackbar.show = true;
        
        // Close dialog
        this.deleteDialog.show = false;
      } catch (error) {
        console.error('Error deleting reader:', error);
        this.snackbar.color = 'error';
        this.snackbar.text = 'Không thể xóa tài khoản. Vui lòng thử lại.';
        this.snackbar.show = true;
      } finally {
        this.deleteDialog.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.readers-page {
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

/* Pagination styling */
:deep(.v-pagination__item--active) {
  background-color: #2e7d32 !important;
  color: white !important;
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
  min-width: 800px; /* Adjust this value as needed */
}

/* Width for specific columns */
.readers-table :deep(table th.column-stt) {
  width: 60px;
  min-width: 60px;
}

.readers-table :deep(table th.column-date) {
  width: 150px;
  min-width: 150px;
}

.readers-table :deep(table th.column-status) {
  width: 120px;
  min-width: 120px;
}

.readers-table :deep(table th.column-actions) {
  width: 150px;
  min-width: 150px;
}
</style>