<template>
  <div>
    <!-- Banner -->
    <div class="position-relative mb-5">
      <v-img src="@/assets/images/banner-background.png" height="600" cover class="relative">
        <div class="banner-overlay position-absolute h-100 w-100"></div>
        <v-container class="fill-height">
          <v-row justify="center">
            <v-col cols="12" md="10" lg="8" style="position: absolute; bottom: 50px;">
              <v-card class="bg-opacity-70 pa-2 d-flex align-center rounded-pill" flat>
                <v-text-field v-model="searchQuery" placeholder="Tìm kiếm sách, Bài báo, Tài liệu..." variant="plain"
                  hide-details class="search-input flex-1 ml-3 no-padding" density="compact"
                  bg-color="transparent"></v-text-field>
                <v-btn color="green" class="search-button rounded-pill" elevation="0" height="45" width="120"
                  @click="search">
                  Tìm kiếm
                </v-btn>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-img>
    </div>

    <div>
      <!-- Những cuốn sách nổi bật -->
      <v-container class="py-8">
        <v-row>
          <v-col cols="12">
            <p class="text-h4 text-left mb-5">NHỮNG CUỐN SÁCH NỔI BẬT</p>
          </v-col>
        </v-row>

        <v-row v-if="loadingFamousBooks">
          <v-col cols="12" class="text-center">
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
          </v-col>
        </v-row>

        <v-row v-else justify="start">
          <v-col v-for="book in mappedFamousBooks" 
                 :key="book.id" 
                 cols="auto"
                 class="ma-2">
            <BookCard :book="book" />
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" class="d-flex justify-end">
            <router-link to="/books" class="text-decoration-none">
              <span class="text-body-2">Hiển thị thêm <v-icon small>mdi-chevron-double-right</v-icon></span>
            </router-link>
          </v-col>
        </v-row>
      </v-container>
    </div>

    <div class="my-10">
      <v-container>
        <p class="text-h4 text-left mb-5">CÓ GÌ TẠI CITY LIBRARY</p>
      </v-container>

      <v-container fluid class="pa-0">
        <v-row no-gutters>
          <v-col cols="12">
            <div class="position-relative">
              <v-img src="@/assets/images/thu-vien.png" height="400" cover>
                <div class="d-flex justify-end align-center h-100">
                  <v-sheet width="500" height="300"
                    class="bg-opacity-70 pa-8 rounded mr-8 d-flex flex-column justify-center">
                    <p class="text-green-darken-2 text-justify" style="font-size: 20px;">
                      City Library là thư viện cung cấp dịch vụ mượn sách với thủ tục nhanh chóng,
                      chính sách mượn – trả linh hoạt, giúp bạn dễ dàng tiếp cận và tận hưởng kho
                      tàng sách phong phú mọi lúc.
                    </p>
                  </v-sheet>
                </div>
              </v-img>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </div>

    <div>
      <v-container class="py-8">
        <v-row>
          <v-col cols="12">
            <p class="text-h4 text-left mb-5">SÁCH MỚI</p>
          </v-col>
        </v-row>

        <v-row v-if="loadingNewBooks">
          <v-col cols="12" class="text-center">
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
          </v-col>
        </v-row>

        <v-row v-else justify="start">
          <v-col v-for="book in mappedNewBooks" 
                 :key="book.id" 
                 cols="auto"
                 class="ma-2">
            <BookCard :book="book" />
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" class="d-flex justify-end">
            <router-link to="/books" class="text-decoration-none">
              <span class="text-body-2">Hiển thị thêm <v-icon small>mdi-chevron-double-right</v-icon></span>
            </router-link>
          </v-col>
        </v-row>
      </v-container>
    </div>

    <!-- Thống kê -->
    <v-container fluid class="statistics-section py-12" style="background-color: #E8F5E9;">
      <v-container>
        <v-row>
          <v-col cols="12" md="6" class="text-center mb-6 mb-md-0">
            <div class="d-flex align-center justify-center my-5">
              <p class="text-h2 font-weight-bold mb-0">1000+</p>
              <p class="text-subtitle-1 mb-0 ml-2">tựa sách</p>
            </div>
            <p class="text-body-1 mt-4">Đa dạng thể loại sách, báo</p>
          </v-col>

          <v-col cols="12" md="6" class="text-center">
            <div class="d-flex align-center justify-center my-5">
              <p class="text-h2 font-weight-bold mb-0">#1</p>
              <p class="text-subtitle-1 mb-0 ml-2">Thư viện online<br>được bạn đọc yêu thích</p>
            </div>
            <p class="text-body-1 mt-4">Thủ tục mượn nhanh gọn</p>
          </v-col>
        </v-row>
      </v-container>
    </v-container>
  </div>
</template>

<script>
import BookCard from '~/components/BookCard.vue'

export default {
  components: {
    BookCard
  },
  
  data() {
    return {
      searchQuery: '',
      famousBooks: [],
      loadingFamousBooks: false,
      newBooks: [],
      loadingNewBooks: false,
    }
  },
  
  computed: {
    // Chuyển đổi dữ liệu từ API sang định dạng cho BookCard component
    mappedFamousBooks() {
      return this.famousBooks.map(book => ({
        id: book.MaSach,
        title: book.TieuDe,
        author: book.TacGia,
        publishYear: book.NamXuatBan,
        publisher: book.NhaXuatBan,
        category: this.getCategoryLabel(book.TheLoai),
        coverImage: book.AnhMinhHoaURL || 'http://www.svgrepo.com/show/508699/landscape-placeholder.svg'
      }));
    },
    
    mappedNewBooks() {
      return this.newBooks.map(book => ({
        id: book.MaSach,
        title: book.TieuDe,
        author: book.TacGia,
        publishYear: book.NamXuatBan,
        publisher: book.NhaXuatBan,
        category: this.getCategoryLabel(book.TheLoai),
        coverImage: book.AnhMinhHoaURL || 'https://www.svgrepo.com/show/508699/landscape-placeholder.svg'
      }));
    }
  },

  mounted() {
    this.getFamousBooks();
    this.getNewBooks();
  },

  methods: {
    search() {
      // Navigate to books page with search query as parameter
      if (this.searchQuery.trim()) {
        this.$router.push({
          path: '/books',
          query: { q: this.searchQuery.trim() }
        });
      } else {
        // If search query is empty, just navigate to books page
        this.$router.push('/books');
      }
    },

    async getFamousBooks() {
      this.loadingFamousBooks = true;
      try {
        const response = await fetch('http://26.193.242.15:8000/books/get_famous_list?page=1&page_size=4', {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
          },
        });
        const data = await response.json();
        this.famousBooks = data || [];
      } catch (error) {
        console.error('Error fetching famous books:', error);
        this.famousBooks = [];
      } finally {
        this.loadingFamousBooks = false;
      }
    },

    async getNewBooks() {
      this.loadingNewBooks = true;
      try {
        const response = await fetch('https://26.193.242.15:8080/books/get_new_list?page=1&page_size=4', {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
          },
        });
        const data = await response.json();
        this.newBooks = data || [];
      } catch (error) {
        console.error('Error fetching new books:', error);
        this.newBooks = [];
      } finally {
        this.loadingNewBooks = false;
      }
    },

    getCategoryLabel(category) {
      const categories = {
        'kinh-te': 'Kinh tế',
        'van-hoc': 'Văn học',
        'tieu-thuyet': 'Tiểu thuyết',
        'ky-nang': 'Kỹ năng',
        'tu-truyen': 'Tự truyện',
      };

      return categories[category] || category || 'Khác';
    }
  }
}
</script>

<style scoped>
.banner-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.3);
}

.bg-opacity-70 {
  background: rgba(255, 255, 255, 0.7) !important;
}

.search-input ::v-deep .v-field__outline {
  display: none;
}

/* Thêm CSS để loại bỏ padding của v-text-field */
.no-padding :deep(.v-field__field) {
  padding-top: 0 !important;
}

.no-padding :deep(.v-field__input) {
  padding-top: 0 !important;
  padding-bottom: 0 !important;
  min-height: 45px !important;
  display: flex;
  align-items: center;
}

.no-padding :deep(.v-field) {
  margin-bottom: 0 !important;
  margin-top: 0 !important;
  padding-top: 0 !important;
}

.position-relative {
  position: relative;
}

.position-absolute {
  position: absolute;
}
</style>