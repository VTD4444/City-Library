<template>
  <div class="manage-books-page">
    <v-container fluid>
      <!-- Filter Tabs -->
      <div class="filters-container mb-4 d-flex flex-wrap align-center">
        <div class="ml-auto mb-2">
          <v-btn-toggle v-model="selectedFilter" mandatory density="comfortable">
            <v-btn value="all" variant="text">Tất cả sách</v-btn>
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
      
      <borrow-tickets-tab 
        v-else-if="selectedFilter === 'expired'"
        :search-query="search"
      ></borrow-tickets-tab>
    </v-container>

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
  }
};
</script>

<style scoped>
.manage-books-page {
  padding-bottom: 40px;
}
</style>