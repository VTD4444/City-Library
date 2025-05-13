<template>
  <div class="borrowed-books-tab">
    <!-- Header Bar with Search -->
    <div class="d-flex flex-wrap align-center justify-space-between mb-4">
      <div class="search-container d-flex ml-auto">
        <v-text-field
          v-model="search"
          placeholder="Nhập mã sách/tên sách/bạn đọc"
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
      <div class="mt-2">Đang tải dữ liệu sách đã mượn...</div>
    </v-card>

    <!-- Error state -->
    <v-alert v-else-if="error" type="error" class="mb-6">
      {{ error }}
    </v-alert>

    <!-- Empty state -->
    <v-card v-else-if="!allBorrowedBooks.length" elevation="0" class="pa-6 text-center">
      <v-icon size="64" color="blue-grey" class="mb-4">mdi-book-arrow-right</v-icon>
      <h2 class="text-h5 mb-2">Không có sách đang mượn</h2>
      <p class="mb-6 text-body-1">
        Hiện tại không có độc giả nào đang mượn sách
      </p>
    </v-card>

    <!-- Borrowed Books Table -->
    <v-card v-else elevation="0">
      <div class="table-responsive">
        <v-data-table
          :headers="tableHeaders"
          :items="paginatedBorrowedBooks"
          class="borrowed-books-table"
          hide-default-footer
        >
          <!-- STT Column -->
          <template v-slot:item.stt="{ item }">
            {{ item.stt }}
          </template>

          <!-- Book ID Column -->
          <template v-slot:item.MaSach="{ item }">
            {{ item.MaSach }}
          </template>

          <!-- Book Title Column -->
          <template v-slot:item.TenSach="{ item }">
            {{ item.TenSach }}
          </template>

          <!-- Image Column -->
          <template v-slot:item.HinhAnh="{ item }">
            <div class="book-image-container">
              <img
                :src="item.HinhAnh || 'https://www.svgrepo.com/show/508699/landscape-placeholder.svg'"
                :alt="item.TenSach"
                class="book-thumbnail"
              />
            </div>
          </template>

          <!-- Borrow Ticket ID Column -->
          <template v-slot:item.TheLoai="{ item }">
            {{ item.TheLoai }}
          </template>

          <!-- Reader Name Column -->
          <template v-slot:item.TenDocGia="{ item }">
            {{ item.HoTen }}
          </template>

          <!-- Borrow Date Column -->
          <template v-slot:item.NgayMuon="{ item }">
            {{ formatDate(item.NgayYeuCau) }}
          </template>

          <!-- Expected Return Date Column -->
          <template v-slot:item.NgayTraDuKien="{ item }">
            {{ formatDate(item.NgayPhaiTra) }}
          </template>

          <!-- Status Column -->
          <template v-slot:item.TrangThai="{ item }">
            <v-select
              v-model="item.TrangThaiSachMuon"
              :items="statusOptions"
              density="compact"
              variant="outlined"
              hide-details
              class="status-select"
              :class="getStatusClass(item.TrangThai)"
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
                @click="viewBookDetail(item.MaSach)"
              >
                Xem
              </v-btn>
              <v-btn
                variant="text"
                color="red"
                size="small"
                class="action-button"
                @click="$emit('delete-borrow', item)"
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

    <!-- Status Update Snackbar -->
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
  name: "BorrowedBooksTab",
  props: {
    searchQuery: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      // Table data
      allBorrowedBooks: [],
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 10,
      search: "", // Local search state
      searchTimeout: null, // For debouncing search
      
      // Status options
      statusOptions: [
        { title: 'Đã mượn', value: 'DaMuon' },
        { title: 'Đã trả', value: 'DaTra' },
      ],
      
      // Table headers
      tableHeaders: [
        { title: "STT", key: "stt", align: "center", width: "50px", sortable: false },
        { title: "Mã sách", key: "MaSach", align: "center", width: "100px" },
        { title: "Tên sách", key: "TenSach", align: "left", width: "200px" },
        { title: "Hình ảnh", key: "HinhAnh", align: "center", width: "100px", sortable: false },
        { title: "Thể Loại", key: "TheLoai", align: "center", width: "130px" },
        { title: "Bạn đọc mượn", key: "TenDocGia", align: "left", width: "150px" },
        { title: "Ngày mượn", key: "NgayMuon", align: "center", width: "120px" },
        { title: "Ngày trả dự kiến", key: "NgayTraDuKien", align: "center", width: "150px" },
        { title: "Trạng thái", key: "TrangThai", align: "center", width: "120px", sortable: false },
        { title: "Hành động", key: "actions", align: "center", width: "120px", sortable: false }
      ],
      
      // Snackbar for notifications
      snackbar: {
        show: false,
        text: "",
        color: "success"
      }
    };
  },
  computed: {
    // Filter borrowed books based on search term
    filteredBorrowedBooks() {
      if (!this.allBorrowedBooks.length) return [];
      
      if (!this.search) {
        return this.allBorrowedBooks;
      }
      
      const searchLower = this.search.toLowerCase().trim();
      return this.allBorrowedBooks.filter(book => 
        (book.MaSach && book.MaSach.toString().toLowerCase().includes(searchLower)) ||
        (book.TenSach && book.TenSach.toLowerCase().includes(searchLower)) ||
        (book.HoTen && book.HoTen.toLowerCase().includes(searchLower))
      );
    },
    
    // Get the paginated borrowed books
    paginatedBorrowedBooks() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      
      return this.filteredBorrowedBooks
        .slice(startIndex, endIndex)
        .map((book, index) => ({
          ...book,
          stt: startIndex + index + 1
        }));
    },
    
    // Calculate total pages
    totalPages() {
      return Math.max(1, Math.ceil(this.filteredBorrowedBooks.length / this.itemsPerPage));
    }
  },
  watch: {
    // Watch for changes in parent's searchQuery to update local search
    searchQuery: {
      handler(newVal) {
        this.search = newVal;
      },
      immediate: true
    },
    
    // Add watcher for local search to trigger API calls
    search(newVal) {
      this.page = 1;
      
      // Clear any existing timeout
      if (this.searchTimeout) {
        clearTimeout(this.searchTimeout);
      }
      
      // Set a timeout to avoid making API calls for every keystroke
      this.searchTimeout = setTimeout(() => {
        this.fetchBorrowedBooks();
      }, 500); // 500ms delay
    }
  },
  mounted() {
    this.fetchBorrowedBooks();
  },
  methods: {
    async fetchBorrowedBooks() {
      this.loading = true;
      this.error = null;

      try {
        // Make API request to get borrowed books
        const response = await fetch("http://26.193.242.15:8000/chitietphieumuon/searchfilterchitietphieumuon", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json"
          },
          body: JSON.stringify({
            MaSachorTieuDeorTenTacGia: this.search.trim(),
            trang_thai: "", // Empty to get all statuses
            page: 1,
            page_size: 1000 // Get a large number to fetch all at once
          })
        });

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể tải danh sách sách đã mượn");
        }

        const data = await response.json();
        this.allBorrowedBooks = Array.isArray(data) ? data : [];
      } catch (error) {
        console.error("Error fetching borrowed books:", error);
        this.error = `Lỗi: ${error.message || "Không thể tải danh sách sách đã mượn"}`;
        this.allBorrowedBooks = [];
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
        year: "numeric"
      });
    },

    getStatusClass(status) {
      switch (status) {
        case 'DaMuon':
          return 'status-borrowed';
        case 'DaTra':
          return 'status-returned';
        default:
          return '';
      }
    },

    async updateStatus(item) {
      try {
        // Make API request to update book borrow status
        const response = await fetch(`http://26.193.242.15:8000/chitietphieumuon/${item.MaPhieuMuon}`, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            TrangThai: item.TrangThai
          })
        });

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể cập nhật trạng thái");
        }

        this.snackbar = {
          show: true,
          text: "Cập nhật trạng thái thành công!",
          color: "success"
        };
      } catch (error) {
        console.error("Error updating status:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể cập nhật trạng thái"}`,
          color: "error"
        };
      }
    },

    viewBorrowDetail(borrowId) {
      // Navigate to borrow detail page or show detail dialog
      this.$emit('view-borrow', borrowId);
    },

    viewBookDetail(bookId) {
      // Navigate to book detail page or show detail dialog
      this.$emit('view-book', bookId);
    }
  }
};
</script>

<style scoped>
.borrowed-books-tab {
  padding-bottom: 40px;
}

/* Table styling similar to genres.vue */
.borrowed-books-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

/* Styling for table */
.borrowed-books-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.borrowed-books-table :deep(th),
.borrowed-books-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

/* Book thumbnail styling */
.book-image-container {
  width: 50px;
  height: 50px;
  margin: 0 auto;
  overflow: hidden;
  border-radius: 4px;
}

.book-thumbnail {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Status select styling */
.status-select {
  min-width: 110px;
}

.status-borrowed :deep(.v-field) {
  background-color: rgba(76, 175, 80, 0.1) !important;
}

.status-returned :deep(.v-field) {
  background-color: rgba(66, 165, 245, 0.1) !important;
}

.status-overdue :deep(.v-field) {
  background-color: rgba(244, 67, 54, 0.1) !important;
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
</style>