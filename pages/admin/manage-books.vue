<template>
  <div class="manage-books-page">
    <v-container fluid>
      <!-- Filter Tabs -->
      <div class="filters-container mb-4 d-flex flex-wrap align-center">
        <div class="ml-auto mb-2">
          <v-btn-toggle v-model="selectedFilter" mandatory density="comfortable">
            <v-btn value="all" variant="text">Tất cả sách</v-btn>
            <v-btn value="borrowed" variant="text">Sách đã cho mượn</v-btn>
            <v-btn value="expired" variant="text">Phiếu mượn</v-btn>
          </v-btn-toggle>
        </div>
      </div>

      <!-- Tab Content - Conditional Rendering -->
      <all-books-tab 
        v-if="selectedFilter === 'all'"
        :search-query="search"
        :selected-filter="selectedFilter"
        :sort-by="sortBy"
        @view-book="viewBookDetail"
        @delete-book="confirmDelete"
      ></all-books-tab>
      
      <borrowed-books-tab 
        v-else-if="selectedFilter === 'borrowed'"
        :search-query="search"
        @view-book="viewBookDetail"
        @delete-book="confirmDelete"
      ></borrowed-books-tab>
      
      <borrow-tickets-tab 
        v-else-if="selectedFilter === 'expired'"
        :search-query="search"
      ></borrow-tickets-tab>
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
          Bạn có chắc chắn muốn xóa sách "{{ deleteDialog.book.TenSach || deleteDialog.book.TieuDe }}"? 
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
import AllBooksTab from '@/components/ManageBooksTabs/AllBooksTab.vue';
import BorrowedBooksTab from '@/components/ManageBooksTabs/BorrowedBooksTab.vue';
import BorrowTicketsTab from '@/components/ManageBooksTabs/BorrowTicketsTab.vue';

export default {
  name: "ManageBooks",
  components: {
    AllBooksTab,
    BorrowedBooksTab,
    BorrowTicketsTab
  },
  data() {
    return {
      // Data for main component
      search: "",
      searchTimeout: null, // For debouncing search
      selectedFilter: "all",
      sortBy: "newest",
      sortOptions: [
        { title: "Ngày tạo", value: "newest" },
        { title: "Tên A-Z", value: "name_asc" },
        { title: "Tên Z-A", value: "name_desc" },
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
          NamXuatBan: new Date().getFullYear(),
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
  watch: {
    // Reset to page 1 when filter changes
    selectedFilter() {
      // This now only handles filter changes at parent level
    },
    // Apply sorting
    sortBy() {
      // This now only handles sort changes at parent level
    }
  },
  methods: {
    viewBookDetail(bookId) {
      this.$router.push(`/admin/book/${bookId}`);
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

    confirmDelete(book) {
      this.deleteDialog = {
        show: true,
        book: book
      };
    },

    async deleteBook() {
      try {
        const response = await fetch(`http://26.193.242.15:8000/books/${this.deleteDialog.book.MaSach}`, {
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
        
        // Force a refetch by changing the search slightly and then back
        const currentSearch = this.search;
        this.search = currentSearch + " ";
        setTimeout(() => {
          this.search = currentSearch;
        }, 100);
      } catch (error) {
        console.error("Error deleting book:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể xóa sách"}`,
          color: "error"
        };
      }
    }
  }
};
</script>

<style scoped>
.manage-books-page {
  padding-bottom: 40px;
}
</style>