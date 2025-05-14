<template>
  <div class="all-books-tab">
    <!-- Header Bar with Search -->
      <div class="d-flex flex-wrap align-center justify-space-between mb-4">
        <div>
          <v-btn
            color="green"
            prepend-icon="mdi-plus"
            @click="goToCreateBookPage"
            class="text-white"
          >
            Thêm sách
          </v-btn>
        </div>

        <div class="search-container d-flex ml-auto">
          <v-text-field
            v-model="search"
            placeholder="Nhập mã sách/tên sách/tác giả"
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
      <div class="mt-2">Đang tải dữ liệu sách...</div>
    </v-card>

    <!-- Error state -->
    <v-alert v-else-if="error" type="error" class="mb-6">
      {{ error }}
    </v-alert>

    <!-- Empty state -->
    <v-card v-else-if="!filteredBooks.length" elevation="0" class="pa-6 text-center">
      <v-icon size="64" color="grey" class="mb-4">mdi-book-off</v-icon>
      <h2 class="text-h5 mb-2">Không tìm thấy sách nào</h2>
      <p class="mb-6 text-body-1">
        Không có sách nào phù hợp với tiêu chí tìm kiếm
      </p>
    </v-card>

    <!-- Books Table -->
    <v-card v-else elevation="0">
      <div class="table-responsive">
        <v-data-table
          :headers="tableHeaders"
          :items="paginatedBooks"
          class="books-table"
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
            {{ item.TieuDe }}
          </template>

          <!-- Image Column -->
          <template v-slot:item.HinhAnh="{ item }">
            <div class="book-image-container">
              <img
                :src="item.AnhMinhHoaURL || '/https://www.svgrepo.com/show/508699/landscape-placeholder.svg'"
                :alt="item.TenSach"
                class="book-thumbnail"
              />
            </div>
          </template>
          
          <!-- Genre Column -->
          <template v-slot:item.TheLoai="{ item }">
            {{ item.TheLoai }}
          </template>

          <!-- Author Column -->
          <template v-slot:item.TacGia="{ item }">
            {{ item.TacGia }}
          </template>

          <!-- Publication Year Column -->
          <template v-slot:item.NamXuatBan="{ item }">
            {{ item.NamXuatBan }}
          </template>

          <!-- Creation Date Column -->
          <template v-slot:item.NgayTao="{ item }">
            {{ formatDate(item.NgayNhap) }}
          </template>

          <!-- Borrowed Quantity Column -->
          <template v-slot:item.SoLuongDaMuon="{ item }">
            {{ item.SoLuotDaMuon }}
          </template>

          <!-- Available Quantity Column -->
          <template v-slot:item.SoLuongConLai="{ item }">
            {{ item.SoLuongHienCo }}
          </template>

          <!-- Total In Document Column -->
          <template v-slot:item.SoLuongTaiLieu="{ item }">
            {{ item.SoLuongTong }}
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
                @click="deleteBook(item)"
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
        
        <span class="mx-2">
          Trang {{ page }} / {{ totalPages }}
        </span>
        
        <v-btn color="green" icon @click="page++" :disabled="page >= totalPages">
          <v-icon>mdi-chevron-right</v-icon>
        </v-btn>
      </div>
    </v-card>

    <!-- Add custom delete confirmation dialog -->
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
          <p class="text-body-1">Bạn có chắc chắn muốn xóa sách này?</p>
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

    <!-- Add snackbar for notifications if not already present -->
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
  name: "AllBooksTab",
  props: {
    searchQuery: {
      type: String,
      default: ""
    },
    selectedFilter: {
      type: String,
      default: "all"
    },
    sortBy: {
      type: String,
      default: "newest"
    }
  },
  data() {
    return {
      // Table data
      allBooks: [],
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 10,
      search: "", // Local search state
      searchTimeout: 500, // For debouncing search
      
      // Table headers
      tableHeaders: [
        { title: "STT", key: "stt", align: "center", width: "50px", sortable: false },
        { title: "Mã sách", key: "MaSach", align: "center", width: "100px" },
        { title: "Tên sách", key: "TenSach", width: "200px" },
        { title: "Hình ảnh", key: "HinhAnh", align: "center", width: "80px", sortable: false },
        { title: "Thể loại", key: "TheLoai", align: "center", sortable: false },
        { title: "Tác giả", key: "TacGia", align: "left" },
        { title: "Năm xuất bản", key: "NamXuatBan", align: "center", width: "120px" },
        { title: "Ngày tạo", key: "NgayTao", align: "center", width: "120px" },
        { title: "SL đã mượn", key: "SoLuongDaMuon", align: "center", width: "90px" },
        { title: "SL còn lại", key: "SoLuongConLai", align: "center", width: "90px" },
        { title: "SL tổng", key: "SoLuongTaiLieu", align: "center", width: "80px" },
        { title: "Hành động", key: "actions", align: "center", width: "120px", sortable: false }
      ],
      deleteDialog: false,
      bookToDelete: null,
      deleteLoading: false,
      snackbar: {
        show: false,
        text: '',
        color: 'success'
      }
    };
  },
  computed: {
    // Filter books based on filters and sorting only (search is done via API)
    filteredBooks() {
      if (!this.allBooks.length) return [];
      
      let result = [...this.allBooks];
      
      // Apply filter type (borrowed, available, etc)
      if (this.selectedFilter !== 'all') {
        if (this.selectedFilter === 'borrowed') {
          result = result.filter(book => book.SoLuotDaMuon > 0);
        } else if (this.selectedFilter === 'available') {
          result = result.filter(book => book.SoLuongHienCo > 0);
        } else if (this.selectedFilter === 'expired') {
          // Implement logic for expired books if needed
        }
      }
      
      // Apply sorting
      if (this.sortBy === 'name_asc') {
        result.sort((a, b) => (a.TieuDe || '').localeCompare(b.TieuDe || ''));
      } else if (this.sortBy === 'name_desc') {
        result.sort((a, b) => (b.TieuDe || '').localeCompare(a.TieuDe || ''));
      } else if (this.sortBy === 'newest') {
        result.sort((a, b) => new Date(b.NgayNhap || 0) - new Date(a.NgayNhap || 0));
      }
      
      return result;
    },
    
    // Get the paginated books
    paginatedBooks() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      
      return this.filteredBooks
        .slice(startIndex, endIndex)
        .map((book, index) => ({
          ...book,
          stt: startIndex + index + 1
        }));
    },
    
    // Calculate total pages
    totalPages() {
      return Math.max(1, Math.ceil(this.filteredBooks.length / this.itemsPerPage));
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
        this.fetchBooks();
      }, 500); // 500ms delay
    }
  },
  mounted() {
    this.fetchBooks();
  },
  methods: {
    async fetchBooks() {
      this.loading = true;
      this.error = null;

      try {
        // Use local search state instead of prop
        const requestBody = {
          TieuDeorMaSachorTenTacGia: this.search.trim(),
          page: 1,
          page_size: 1000 // Get a large number to handle client-side filtering and pagination
        };
        
        // Make API request
        const response = await fetch(`http://26.193.242.15:8000/books/thongke_sach_filter`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json"
          },
          body: JSON.stringify(requestBody)
        });
        
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể tải danh sách sách");
        }
        
        const data = await response.json();
        this.allBooks = Array.isArray(data.books) ? data.books : (Array.isArray(data) ? data : []);
      } catch (error) {
        console.error("Error fetching books:", error);
        this.error = `Lỗi: ${error.message || "Không thể tải danh sách sách"}`;
        this.allBooks = [];
      } finally {
        this.loading = false;
      }
    },
    
    async saveBook() {
      if (!this.$refs.bookForm.validate()) return;

      try {
        // Prepare form data for API request (including file upload)
        const formData = new FormData();
        
        // Add book data
        Object.keys(this.bookDialog.book).forEach(key => {
          if (key !== "HinhAnhFile") {
            formData.append(key, this.bookDialog.book[key]);
          }
        });
        
        // Add image file if selected
        if (this.bookDialog.book.HinhAnhFile) {
          formData.append("HinhAnh", this.bookDialog.book.HinhAnhFile);
        }

        let response;
        if (this.bookDialog.isEdit) {
          // Update existing book
          formData.append("MaSach", this.bookDialog.book.MaSach);
          response = await fetch(`http://26.193.242.15:8000/books/${this.bookDialog.book.MaSach}`, {
            method: "PUT",
            body: formData,
          });
        } else {
          // Create new book
          response = await fetch("http://26.193.242.15:8000/books", {
            method: "POST",
            body: formData,
          });
        }

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể lưu thông tin sách");
        }

        // Success handling
        this.snackbar = {
          show: true,
          text: this.bookDialog.isEdit ? "Cập nhật sách thành công!" : "Thêm sách mới thành công!",
          color: "success"
        };
        
        this.closeBookDialog();
        // Force a refetch by changing the search slightly and then back
        const currentSearch = this.search;
        this.search = currentSearch + " ";
        setTimeout(() => {
          this.search = currentSearch;
        }, 100);
      } catch (error) {
        console.error("Error saving book:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể lưu thông tin sách"}`,
          color: "error"
        };
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

    goToCreateBookPage() {
      this.$router.push("/admin/book/create");
    },

    viewBookDetail(bookId) {
      this.$router.push(`/admin/book/${bookId}`);
    },
    
    deleteBook(book) {
      // Set the book to delete and show the confirmation dialog
      this.bookToDelete = book;
      this.deleteDialog = true;
    },
    
    async confirmDelete() {
      if (!this.bookToDelete) return;
      
      this.deleteLoading = true;
      try {
        // Send delete request to API
        const response = await fetch(`http://26.193.242.15:8000/booksdelete/${this.bookToDelete.MaSach}`, {
          method: 'DELETE',
          headers: {
            'Accept': 'application/json'
          }
        });
        
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || 'Không thể xóa sách');
        }
        
        // Show success message
        this.snackbar = {
          show: true,
          text: 'Sách đã được xóa thành công',
          color: 'success'
        };
        
        // Close the dialog and refresh the list
        this.deleteDialog = false;
        this.bookToDelete = null;
        
        // Refresh the list of books
        await this.fetchBooks();
        
      } catch (error) {
        console.error('Error deleting book:', error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || 'Không thể xóa sách'}`,
          color: 'error'
        };
      } finally {
        this.deleteLoading = false;
      }
    }
  }
};
</script>

<style scoped>
.books-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

/* Styling for table */
.books-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.books-table :deep(th),
.books-table :deep(td) {
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

/* Table responsive container */
.table-responsive {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

.search-container {
  max-width: 500px;
}

.search-field {
  min-width: 300px;
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

@media (max-width: 600px) {
  .search-field {
    min-width: 200px;
  }
}
</style>