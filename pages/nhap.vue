<template>
  <div class="manage-books-page">
    <v-container fluid>
      <!-- Header Bar with Search -->
      <div class="d-flex flex-wrap align-center justify-space-between mb-4">
        <div>
          <v-btn
            color="green"
            prepend-icon="mdi-plus"
            @click="openAddBookDialog"
            class="text-white rounded-pill"
            size="small"
          >
            Thêm sách
          </v-btn>
        </div>

        <div class="d-flex align-center ms-auto">
          <v-text-field
            v-model="search"
            placeholder="Nhập từ khóa tìm kiếm"
            density="compact"
            variant="outlined"
            hide-details
            append-inner-icon="mdi-magnify"
            single-line
            class="search-field rounded-pill mx-2"
            bg-color="white"
          ></v-text-field>
          
          <v-menu>
            <template v-slot:activator="{ props }">
              <v-btn
                v-bind="props"
                variant="outlined"
                color="black"
                size="small"
                class="ms-2"
              >
                Ngày trả <v-icon>mdi-chevron-down</v-icon>
              </v-btn>
            </template>
            <v-list>
              <v-list-item value="all">Tất cả</v-list-item>
              <v-list-item value="upcoming">Sắp tới</v-list-item>
              <v-list-item value="past">Đã qua</v-list-item>
            </v-list>
          </v-menu>
        </div>
      </div>

      <!-- Navigation Tabs -->
      <v-card flat class="mb-4">
        <v-tabs
          v-model="selectedTab"
          bg-color="white"
          color="green"
        >
          <v-tab value="all">Tất cả sách</v-tab>
          <v-tab value="borrowed">Sách đã cho mượn</v-tab>
          <v-tab value="reserved">Sách đã được giữ</v-tab>
          <v-tab value="tickets">Phiếu mượn</v-tab>
          <v-tab value="returns">Ngày trả</v-tab>
        </v-tabs>
      </v-card>

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
                  :src="item.HinhAnh ? item.HinhAnh : '/https://www.svgrepo.com/show/508699/landscape-placeholder.svg'"
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
                  @click="viewBookDetail(item.MaSach)"
                >
                  Xem
                </v-btn>
                <v-btn
                  variant="text"
                  color="red"
                  size="small"
                  @click="confirmDelete(item)"
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
    </v-container>

    <!-- Add/Edit Book Dialog -->
    <v-dialog v-model="bookDialog.show" max-width="800px">
      <v-card>
        <v-card-title class="text-h5">
          {{ bookDialog.isEdit ? 'Chỉnh sửa sách' : 'Thêm sách mới' }}
        </v-card-title>
        
        <v-card-text>
          <v-form ref="bookForm" v-model="bookDialog.valid">
            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="bookDialog.book.TenSach"
                  label="Tên sách"
                  required
                  :rules="[v => !!v || 'Tên sách là bắt buộc']"
                ></v-text-field>
              </v-col>
              
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="bookDialog.book.TacGia"
                  label="Tác giả"
                  required
                  :rules="[v => !!v || 'Tác giả là bắt buộc']"
                ></v-text-field>
              </v-col>
              
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="bookDialog.book.TheLoai"
                  label="Thể loại"
                  required
                  :rules="[v => !!v || 'Thể loại là bắt buộc']"
                ></v-text-field>
              </v-col>
              
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="bookDialog.book.NamXuatBan"
                  label="Năm xuất bản"
                  type="number"
                  required
                  :rules="[v => !!v || 'Năm xuất bản là bắt buộc',
                    v => (parseInt(v) >= 1800 && parseInt(v) <= new Date().getFullYear()) || 'Năm xuất bản không hợp lệ']"
                ></v-text-field>
              </v-col>
              
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="bookDialog.book.SoLuong"
                  label="Số lượng"
                  type="number"
                  required
                  :rules="[v => !!v || 'Số lượng là bắt buộc',
                    v => parseInt(v) > 0 || 'Số lượng phải lớn hơn 0']"
                ></v-text-field>
              </v-col>
              
              <v-col cols="12">
                <v-file-input
                  v-model="bookDialog.book.HinhAnhFile"
                  label="Hình ảnh sách"
                  accept="image/*"
                  prepend-icon="mdi-camera"
                  show-size
                  :rules="[v => !v || v.size < 2000000 || 'Kích thước hình ảnh phải nhỏ hơn 2MB']"
                ></v-file-input>
              </v-col>
              
              <v-col cols="12">
                <v-textarea
                  v-model="bookDialog.book.MoTa"
                  label="Mô tả"
                  rows="3"
                ></v-textarea>
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>
        
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="grey" variant="text" @click="closeBookDialog">
            Hủy
          </v-btn>
          <v-btn color="green-darken-1" @click="saveBook">
            {{ bookDialog.isEdit ? 'Cập nhật' : 'Thêm sách' }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Delete Confirmation Dialog -->
    <v-dialog v-model="deleteDialog.show" max-width="400">
      <v-card>
        <v-card-title class="text-h5">Xóa sách?</v-card-title>
        <v-card-text>
          Bạn có chắc chắn muốn xóa sách "{{ deleteDialog.book.TenSach }}"? 
          Hành động này không thể hoàn tác.
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="grey" variant="text" @click="deleteDialog.show = false">
            Hủy
          </v-btn>
          <v-btn color="error" @click="deleteBook">
            Xóa sách
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Snackbar Notification -->
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
  name: "ManageBooks",
  data() {
    return {
      // Add new tab selection variable
      selectedTab: "all",
      
      // Rest of the data properties remain the same
      allBooks: [],
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 10,
      search: "",
      searchTimeout: null,
      selectedFilter: "all",
      sortBy: "newest",
      sortOptions: [
        { title: "Ngày tạo", value: "newest" },
        { title: "Tên A-Z", value: "name_asc" },
        { title: "Tên Z-A", value: "name_desc" },
      ],

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

      // Book dialog
      bookDialog: {
        show: false,
        isEdit: false,
        valid: true,
        book: {
          TenSach: "",
          TacGia: "",
          TheLoai: "",
          NamXuatBan: "",
          SoLuong: 1,
          MoTa: "",
          HinhAnhFile: null
        }
      },

      // Delete dialog
      deleteDialog: {
        show: false,
        book: {}
      },

      // Notification
      snackbar: {
        show: false,
        text: "",
        color: "success"
      }
    };
  },
  
  // Watch for changes in the selected tab
  watch: {
    selectedTab(newValue) {
      // Map tab value to filter value
      this.selectedFilter = newValue;
      this.page = 1;
    },
    
    // Keep existing watchers
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
    },
    
    // Reset to page 1 when filter changes
    selectedFilter() {
      this.page = 1;
    },
    // Apply sorting
    sortBy() {
      this.page = 1;
    }
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
  mounted() {
    this.fetchBooks();
  },
  methods: {
    async fetchBooks() {
      this.loading = true;
      this.error = null;

      try {
        // Prepare request body with search term
        const requestBody = {
          TieuDeorMaSachorTenTacGia: this.search.trim(),
          page: 1,
          page_size: 1000 // Get a large number to handle client-side filtering and pagination
        };
        
        // Make API request
        const response = await fetch(`https://26.193.242.15:8080/books/thongke_sach_filter`, {
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

    formatDate(dateString) {
      if (!dateString) return "N/A";
      
      const date = new Date(dateString);
      return date.toLocaleDateString("vi-VN", {
        day: "2-digit",
        month: "2-digit",
        year: "numeric"
      });
    },

    viewBookDetail(bookId) {
      this.$router.push(`/admin/books/${bookId}`);
    },

    openAddBookDialog() {
      this.bookDialog = {
        show: true,
        isEdit: false,
        valid: true,
        book: {
          TenSach: "",
          TacGia: "",
          TheLoai: "",
          NamXuatBan: new Date().getFullYear(),
          SoLuong: 1,
          MoTa: "",
          HinhAnhFile: null
        }
      };
    },

    closeBookDialog() {
      this.bookDialog.show = false;
    },

    async saveBook() {
      if (!this.$refs.bookForm.validate()) return;

      this.loading = true;
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
          response = await fetch(`https://26.193.242.15:8080/books/${this.bookDialog.book.MaSach}`, {
            method: "PUT",
            body: formData,
          });
        } else {
          // Create new book
          response = await fetch("https://26.193.242.15:8080/books", {
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
        this.fetchBooks(); // Refresh all books
      } catch (error) {
        console.error("Error saving book:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể lưu thông tin sách"}`,
          color: "error"
        };
      } finally {
        this.loading = false;
      }
    },

    confirmDelete(book) {
      this.deleteDialog = {
        show: true,
        book: book
      };
    },

    async deleteBook() {
      this.loading = true;
      try {
        const response = await fetch(`https://26.193.242.15:8080/books/${this.deleteDialog.book.MaSach}`, {
          method: "DELETE"
        });

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể xóa sách");
        }

        this.snackbar = {
          show: true,
          text: "Xóa sách thành công!",
          color: "success"
        };
        
        this.deleteDialog.show = false;
        this.fetchBooks(); // Refresh all books
      } catch (error) {
        console.error("Error deleting book:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể xóa sách"}`,
          color: "error"
        };
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
/* Add these styles to match the design */
.search-field {
  min-width: 300px;
  background-color: white;
}

/* Keep existing styles */
.manage-books-page {
  padding-bottom: 40px;
}

/* Update search container for new layout */
.search-container {
  max-width: unset;
}

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

/* Pagination styling */
:deep(.v-pagination__item--active) {
  background-color: #2e7d32 !important;
  color: white !important;
}

/* Highlight sort active state */
.sort-select :deep(.v-select__selection) {
  color: #2e7d32;
  font-weight: 500;
}
</style>