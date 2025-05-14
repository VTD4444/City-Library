<template>
  <div class="genres-page">
    <v-container>
      <h1 class="text-h4 text-center font-weight-bold mb-6">
        QUẢN LÝ THỂ LOẠI
      </h1>

      <!-- Header and Actions Row -->
      <div class="d-flex flex-wrap align-center justify-space-between mb-4">
        <!-- Right side: Search Bar -->
        <div class="search-container d-flex ml-auto">
          <v-text-field
            v-model="search"
            placeholder="Tìm kiếm thể loại"
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
        <div class="mt-2">Đang tải dữ liệu thể loại...</div>
      </v-card>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Empty state -->
      <v-card
        v-else-if="!allGenres.length"
        elevation="0"
        class="pa-6 text-center"
      >
        <v-icon size="64" color="grey" class="mb-4"
          >mdi-book-variant-multiple</v-icon
        >
        <h2 class="text-h5 mb-2">Không có thể loại nào</h2>
        <p class="mb-6 text-body-1">
          Hiện tại chưa có thể loại nào trong hệ thống
        </p>
      </v-card>

      <!-- Genres Table -->
      <v-card v-else elevation="0" max-width="900px" class="mx-auto">
        <div class="table-responsive">
          <v-data-table
            :headers="tableHeaders"
            :items="paginatedGenres"
            class="genres-table"
            hide-default-footer
          >
            <!-- STT Column -->
            <template v-slot:item.stt="{ item }">
              {{ item.stt }}
            </template>

            <!-- Genre Name Column -->
            <template v-slot:item.TenTheLoai="{ item }">
              {{ item.TenTheLoai }}
            </template>

            <!-- Book Count Column -->
            <template v-slot:item.SoSach="{ item }">
              <span>{{ item.SoSach || 0 }}</span>
            </template>
          </v-data-table>
        </div>

        <!-- Custom Pagination -->
        <div class="d-flex justify-center align-center pa-4">
          <v-btn
            color="white"
            elevation="0"
            icon
            @click="page > 1 ? page-- : page"
          >
            <v-icon color="green">mdi-chevron-left</v-icon>
          </v-btn>
          <span class="mx-2"> Trang {{ page }} / {{ totalPages }} </span>
          <v-btn
            color="white"
            elevation="0"
            icon
            @click="page < totalPages ? page++ : page"
          >
            <v-icon color="green">mdi-chevron-right</v-icon>
          </v-btn>
        </div>
      </v-card>
    </v-container>
  </div>
</template>

<script>
export default {
  name: "GenresPage",
  data() {
    return {
      // Table data
      allGenres: [], // Store all genres
      loading: false,
      error: null,
      page: 1,
      itemsPerPage: 1000,
      search: "",

      // Table headers
      tableHeaders: [
        {
          title: "STT",
          key: "stt",
          align: "center",
          width: "80px",
          sortable: false,
        },
        { title: "Tên thể loại", key: "TenTheLoai", align: "left" },
        { title: "Số sách", key: "SoSach", align: "center", width: "150px" },
      ],
    };
  },
  computed: {
    // Filter genres based on search term
    filteredGenres() {
      if (!this.search) {
        return this.allGenres;
      }

      const searchLower = this.search.toLowerCase().trim();
      return this.allGenres.filter((genre) =>
        genre.TenTheLoai.toLowerCase().includes(searchLower)
      );
    },

    // Get the paginated genres
    paginatedGenres() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;

      return this.filteredGenres
        .slice(startIndex, endIndex)
        .map((genre, index) => ({
          ...genre,
          stt: startIndex + index + 1,
        }));
    },

    // Calculate total pages
    totalPages() {
      return Math.ceil(this.filteredGenres.length / this.itemsPerPage);
    },
  },
  mounted() {
    this.fetchAllGenres();
  },
  watch: {
    // Reset to page 1 when search changes
    search() {
      this.page = 1;
    },
  },
  methods: {
    async fetchAllGenres() {
      this.loading = true;
      this.error = null;

      try {
        // Make API request to get all genres at once
        const response = await fetch(
          "https://26.193.242.15:8080/theloai/thongke-theloai",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Accept: "application/json",
            },
            body: JSON.stringify({
              sort_by: "TheLoai",
              page: 1,
              page_size: 1000, // Get a large number to fetch all at once
            }),
          }
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(
            errorData.message || "Không thể tải danh sách thể loại"
          );
        }

        const data = await response.json();
        this.allGenres = data;
      } catch (error) {
        console.error("Error fetching genres:", error);
        this.error = `Lỗi: ${
          error.message || "Không thể tải danh sách thể loại"
        }`;
        this.allGenres = [];
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.genres-page {
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

.genres-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

/* Styling for table */
.genres-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.genres-table :deep(th),
.genres-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

/* Pagination styling */
:deep(.v-pagination__item--active) {
  background-color: #2e7d32 !important;
  color: white !important;
}

/* Animation for table rows */
.genres-table :deep(tbody tr) {
  transition: background-color 0.2s ease;
}

.genres-table :deep(tbody tr:hover) {
  background-color: rgba(76, 175, 80, 0.05);
}

/* Table responsive container */
.table-responsive {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

/* Set minimum width for the table */
.genres-table {
  min-width: 600px; /* Adjust this value as needed */
}

/* Width for specific columns */
.genres-table :deep(table th.column-stt) {
  width: 80px;
  min-width: 80px;
}

.genres-table :deep(table th.column-book-count) {
  width: 150px;
  min-width: 150px;
}

.genres-table :deep(table th.column-date) {
  width: 150px;
  min-width: 150px;
}
</style>
