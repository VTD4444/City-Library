<template>
  <div class="books-page">
    <v-container fluid>
      <v-row>
        <!-- Filter sidebar -->
        <v-col cols="12" md="3" lg="3">
          <v-card class="pa-4 filter-card" flat>
            <div class="mb-3">
              <label class="text-body-2 mb-1 d-block">Thể loại</label>
              <v-text-field
                v-model="filters.theLoai"
                placeholder="Nhập thể loại"
                variant="outlined"
                density="compact"
                hide-details
                class="mb-3"
              ></v-text-field>
            </div>
            
            <div class="mb-3">
              <label class="text-body-2 mb-1 d-block">Tên sách</label>
              <v-text-field
                v-model="filters.tenSach"
                placeholder="Nhập tên sách"
                variant="outlined"
                density="compact"
                hide-details
                class="mb-3"
              ></v-text-field>
            </div>
            
            <div class="mb-3">
              <label class="text-body-2 mb-1 d-block">Năm xuất bản</label>
              <v-text-field
                v-model="filters.namXuatBan"
                placeholder="Nhập năm xuất bản"
                variant="outlined"
                density="compact"
                hide-details
                class="mb-3"
              ></v-text-field>
            </div>
            
            <div class="mb-3">
              <label class="text-body-2 mb-1 d-block">Tác giả</label>
              <v-text-field
                v-model="filters.tacGia"
                placeholder="Nhập tên tác giả"
                variant="outlined"
                density="compact"
                hide-details
                class="mb-3"
              ></v-text-field>
            </div>
            
            <div class="mb-4">
              <label class="text-body-2 mb-1 d-block">Sắp xếp</label>
              <v-select
                v-model="filters.sortBy"
                :items="sortOptions"
                variant="outlined"
                density="compact"
                hide-details
                class="mb-3"
              ></v-select>
            </div>
            
            <v-btn 
              color="green-darken-1" 
              class="px-6 white--text"
              block
              @click="applyFilters"
            >
              <v-icon left>mdi-magnify</v-icon>
              Tìm kiếm
            </v-btn>
          </v-card>
        </v-col>
        
        <!-- Books grid -->
        <v-col cols="12" md="9" lg="9">
          <v-row v-if="loading" justify="center" class="my-10">
            <v-progress-circular indeterminate color="green" size="64"></v-progress-circular>
          </v-row>
          
          <v-alert 
            v-if="error" 
            type="error" 
            class="mb-4"
            closable
          >
            {{ error }}
          </v-alert>
          
          <div v-if="!loading && !books.length" class="text-center py-10">
            <v-icon size="64" color="grey">mdi-book-search</v-icon>
            <h3 class="mt-4 text-h5 font-weight-medium">Không tìm thấy sách nào</h3>
            <p class="mt-2 text-body-1">Vui lòng thử lại với các tiêu chí tìm kiếm khác</p>
          </div>
          
          <v-row v-if="!loading && books.length">
            <v-col 
              v-for="book in books" 
              :key="book.MaSach" 
              cols="12" sm="6" md="4" lg="3" xl="2"
            >
              <book-card :book="book"></book-card>
            </v-col>
          </v-row>
          
          <!-- Pagination -->
          <div class="d-flex justify-center mt-6">
            <v-pagination
              v-if="totalPages > 1"
              v-model="currentPage"
              :length="totalPages"
              rounded
              color="green-darken-1"
              @update:modelValue="handlePageChange"
            ></v-pagination>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import BookCard from '@/components/BookCard.vue';

export default {
  name: 'BooksPage',
  components: {
    BookCard
  },
  data() {
    return {
      books: [],
      loading: true,
      error: null,
      currentPage: 1,
      totalPages: 0,
      totalBooks: 0,
      filters: {
        theLoai: '',
        tenSach: '',
        namXuatBan: '',
        tacGia: '',
        sortBy: ''
      },
      sortOptions: [
        { title: 'Mới nhất', value: 'newest' },
        { title: 'Cũ nhất', value: 'oldest' },
        { title: 'Tên A-Z', value: 'asc' },
        { title: 'Tên Z-A', value: 'desc' }
      ]
    }
  },
  mounted() {
    // Check if there's a search query in URL
    if (this.$route.query.q) {
      this.filters.tenSach = this.$route.query.q;
    }
    this.fetchBooks();
  },
  methods: {
    async fetchBooks() {
      this.loading = true;
      this.error = null;
      
      try {
        // Tạo request body theo format mới
        const requestBody = {
          "TieuDe": this.filters.tenSach || null,
          "TheLoai": this.filters.theLoai || null,
          "TacGia": this.filters.tacGia || null,
          "NamXuatBan": this.filters.namXuatBan ? parseInt(this.filters.namXuatBan) : 0,
          "sort_by": "title", // Mặc định sắp xếp theo tiêu đề
          "sort_order": "asc"  // Mặc định sắp xếp tăng dần
        };
        
        // Cập nhật sort_by và sort_order dựa vào lựa chọn người dùng
        switch(this.filters.sortBy) {
          case 'newest':
            requestBody.sort_by = "NamXuatBan";
            requestBody.sort_order = "desc";
            break;
          case 'oldest':
            requestBody.sort_by = "NamXuatBan";
            requestBody.sort_order = "asc";
            break;
          case 'asc':
            requestBody.sort_by = "TieuDe";
            requestBody.sort_order = "asc";
            break;
          case 'desc':
            requestBody.sort_by = "TieuDe";
            requestBody.sort_order = "desc";
            break;
        }
        
        // Gọi API mới với phương thức POST
        const response = await fetch(`http://26.193.242.15:8000/books/search-books/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
          },
          body: JSON.stringify(requestBody)
        });
        
        if (!response.ok) {
          throw new Error('Không thể lấy danh sách sách. Vui lòng thử lại sau.');
        }
        
        const data = await response.json();
        
        // Update data - điều chỉnh dựa trên cấu trúc phản hồi của API mới
        this.books = Array.isArray(data) ? data : (data.books || []);
        this.totalBooks = data.total || this.books.length;
        this.totalPages = data.pages || Math.ceil(this.totalBooks / 12);
        
      } catch (error) {
        console.error('Error fetching books:', error);
        this.error = error.message || 'Đã xảy ra lỗi khi tải danh sách sách';
      } finally {
        this.loading = false;
      }
    },
    
    applyFilters() {
      this.currentPage = 1; // Reset về trang đầu tiên khi áp dụng bộ lọc mới
      this.fetchBooks();
    },
    
    handlePageChange(page) {
      this.currentPage = page;
      this.fetchBooks();
      
      // Scroll lên đầu danh sách sách
      window.scrollTo({
        top: 0,
        behavior: 'smooth'
      });
    }
  }
}
</script>

<style scoped>
.books-page {
  padding-bottom: 40px;
}

.filter-card {
  position: sticky;
  top: 20px;
}

@media (max-width: 959px) {
  .filter-card {
    position: relative;
    top: 0;
    margin-bottom: 20px;
  }
}
</style>