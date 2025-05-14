<template>
  <div class="authors-page">
    <v-container>
      <h1 class="text-h4 text-center font-weight-bold mb-6">QUẢN LÝ TÁC GIẢ</h1>

      <!-- Header and Actions Row -->
      <div class="d-flex flex-wrap align-center justify-space-between mb-4">
        <!-- Right side: Search Bar -->
        <div class="search-container d-flex ml-auto">
          <v-text-field
            v-model="search"
            placeholder="Tìm kiếm tác giả"
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
        <div class="mt-2">Đang tải dữ liệu tác giả...</div>
      </v-card>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Empty state -->
      <v-card v-else-if="!allAuthors.length" elevation="0" class="pa-6 text-center">
        <v-icon size="64" color="grey" class="mb-4">mdi-account-group</v-icon>
        <h2 class="text-h5 mb-2">Không có tác giả nào</h2>
        <p class="mb-6 text-body-1">
          Hiện tại chưa có tác giả nào trong hệ thống
        </p>
      </v-card>

      <!-- Authors Table -->
      <v-card v-else elevation="0" max-width="900px" class="mx-auto">
        <div class="table-responsive">
          <v-data-table
            :headers="tableHeaders"
            :items="paginatedAuthors"
            class="authors-table"
            hide-default-footer
          >
            <!-- STT Column -->
            <template v-slot:item.stt="{ item }">
              {{ item.stt }}
            </template>

            <!-- Author Name Column -->
            <template v-slot:item.TenTacGia="{ item }">
              {{ item.TenTacGia }}
            </template>

            <!-- Book Count Column -->
            <template v-slot:item.SoLuongSach="{ item }">
              <span>{{ item.SoLuongSach || 0 }}</span>
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
  </div>
</template>

<script>
export default {
  name: "AuthorsPage",
  data() {
    return {
      // Table data
      allAuthors: [], // Store all authors
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 10, // Display 10 items per page
      search: "",

      // Table headers
      tableHeaders: [
        { title: "STT", key: "stt", align: "center", width: "80px", sortable: false },
        { title: "Tên tác giả", key: "TenTacGia", align: "left" },
        { title: "Số sách", key: "SoLuongSach", align: "center", width: "150px" },
      ],
    };
  },
  computed: {
    // Filter authors based on search term
    filteredAuthors() {
      if (!this.search) {
        return this.allAuthors;
      }
      
      const searchLower = this.search.toLowerCase().trim();
      return this.allAuthors.filter(author => 
        author.TenTacGia.toLowerCase().includes(searchLower)
      );
    },
    
    // Get the paginated authors
    paginatedAuthors() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      
      return this.filteredAuthors
        .slice(startIndex, endIndex)
        .map((author, index) => ({
          ...author,
          stt: startIndex + index + 1
        }));
    },
    
    // Calculate total pages
    totalPages() {
      return Math.ceil(this.filteredAuthors.length / this.itemsPerPage);
    }
  },
  mounted() {
    this.fetchAllAuthors();
  },
  watch: {
    // Reset to page 1 when search changes
    search() {
      this.page = 1;
    }
  },
  methods: {
    async fetchAllAuthors() {
      this.loading = true;
      this.error = null;

      try {
        // Make API request to get all authors at once
        const response = await fetch("https://26.193.242.15:8080/tacgia/thongke-tacgia", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json"
          },
          body: JSON.stringify({
            page: 1,
            page_size: 1000 // Get a large number to fetch all at once
          })
        });

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể tải danh sách tác giả");
        }

        const data = await response.json();
        this.allAuthors = data.items || data;
      } catch (error) {
        console.error("Error fetching authors:", error);
        this.error = `Lỗi: ${error.message || "Không thể tải danh sách tác giả"}`;
        this.allAuthors = [];
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.authors-page {
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

.authors-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

/* Styling for table */
.authors-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.authors-table :deep(th),
.authors-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

/* Pagination styling */
:deep(.v-pagination__item--active) {
  background-color: #2e7d32 !important;
  color: white !important;
}

/* Animation for table rows */
.authors-table :deep(tbody tr) {
  transition: background-color 0.2s ease;
}

.authors-table :deep(tbody tr:hover) {
  background-color: rgba(76, 175, 80, 0.05);
}

/* Table responsive container */
.table-responsive {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

/* Set minimum width for the table */
.authors-table {
  min-width: 600px; /* Adjust this value as needed */
}

/* Width for specific columns */
.authors-table :deep(table th.column-stt) {
  width: 80px;
  min-width: 80px;
}

.authors-table :deep(table th.column-book-count) {
  width: 150px;
  min-width: 150px;
}
</style>