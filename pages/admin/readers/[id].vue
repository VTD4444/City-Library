<template>
  <div class="reader-detail-page">
    <v-container>
      <!-- Back button -->
      <div class="d-flex align-center mb-4">
        <v-btn
          prepend-icon="mdi-arrow-left"
          variant="text"
          @click="$router.back()"
          class="pl-0"
        >
          Quay lại danh sách
        </v-btn>
      </div>

      <!-- Page title -->
      <h1 class="text-h4 text-center font-weight-bold mb-6">
        THÔNG TIN CHI TIẾT BẠN ĐỌC
      </h1>

      <!-- Loading state -->
      <v-row v-if="loading" justify="center" class="my-10">
        <v-progress-circular indeterminate color="primary"></v-progress-circular>
        <div class="mt-2">Đang tải thông tin bạn đọc...</div>
      </v-row>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Reader information -->
      <v-card v-else class="mx-auto" max-width="900px" elevation="0">
        <v-card-text>
          <!-- Header with reader name and actions -->
          <div class="d-flex flex-column flex-sm-row justify-space-between align-start mb-6">
            <div>
              <h2 class="text-h4 font-weight-medium text-green-darken-2">
                {{ reader.HoTen }}
              </h2>
              <div class="text-subtitle-1 text-grey">
                Mã bạn đọc: {{ reader.MaDocGia }}
              </div>
            </div>
            
            <div class="mt-3 mt-sm-0 d-flex">
              <v-btn
                color="error"
                variant="a"
                class="ml-2"
                prepend-icon="mdi-delete"
                @click="confirmDelete"
              >
                Xóa bạn đọc
              </v-btn>
            </div>
          </div>
          
          <!-- Reader information sections -->
          <v-row>
            <v-col cols="12" sm="6">
              <v-card variant="flat" class="mb-4">
                <v-card-title class="text-h6 bg-grey-lighten-4 py-3">
                  Thông tin cá nhân
                </v-card-title>
                <v-card-text class="pt-4">
                  <div class="d-flex mb-4">
                    <div class="info-label">Ngày sinh:</div>
                    <div class="info-value">{{ formatDate(reader.NgaySinh) }}</div>
                  </div>
                  
                  <div class="d-flex mb-4">
                    <div class="info-label">Địa chỉ:</div>
                    <div class="info-value">{{ reader.DiaChi || 'Chưa cung cấp' }}</div>
                  </div>
                  
                  <div class="d-flex mb-4">
                    <div class="info-label">Email:</div>
                    <div class="info-value">{{ reader.Email }}</div>
                  </div>
                  
                  <div class="d-flex mb-4">
                    <div class="info-label">Điện thoại:</div>
                    <div class="info-value">{{ reader.SoDienThoai || 'Chưa cung cấp' }}</div>
                  </div>
                </v-card-text>
              </v-card>
            </v-col>
            
            <v-col cols="12" sm="6">
              <v-card variant="flat" class="mb-4">
                <v-card-title class="text-h6 bg-grey-lighten-4 py-3">
                  Thông tin tài khoản
                </v-card-title>
                <v-card-text class="pt-4">
                  <div class="d-flex mb-4">
                    <div class="info-label">Tên đăng nhập:</div>
                    <div class="info-value">{{ reader.TenDangNhap }}</div>
                  </div>
                  
                  <div class="d-flex mb-4">
                    <div class="info-label">Ngày đăng ký:</div>
                    <div class="info-value">{{ formatDate(reader.NgayDangKy) }}</div>
                  </div>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
          
          <!-- Borrowing statistics -->
          <!-- <v-card variant="flat" class="mb-4">
            <v-card-title class="text-h6 bg-grey-lighten-4 py-3">
              Thống kê mượn trả
            </v-card-title>
            <v-card-text class="pt-4 pb-4">
              <v-row>
                <v-col cols="12" sm="4">
                  <div class="stat-box text-center pa-4">
                    <div class="stat-value green--text text--darken-1">{{ reader.SachDaMuon || 0 }}</div>
                    <div class="stat-label">Sách đã mượn</div>
                  </div>
                </v-col>
                
                <v-col cols="12" sm="4">
                  <div class="stat-box text-center pa-4">
                    <div class="stat-value orange--text text--darken-1">{{ reader.SachDangMuon || 0 }}</div>
                    <div class="stat-label">Đang mượn</div>
                  </div>
                </v-col>
                
                <v-col cols="12" sm="4">
                  <div class="stat-box text-center pa-4">
                    <div class="stat-value red--text text--darken-1">{{ reader.SachQuaHan || 0 }}</div>
                    <div class="stat-label">Quá hạn</div>
                  </div>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card> -->
        </v-card-text>
      </v-card>
      
      <!-- Delete confirmation dialog -->
      <v-dialog v-model="deleteDialog.show" max-width="500px" persistent>
        <v-card>
          <v-card-title class="bg-grey-lighten-3 d-flex align-center">
            <span>Xác nhận xóa bạn đọc</span>
            <v-spacer></v-spacer>
            <v-btn elevation="0" icon @click="deleteDialog.show = false">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </v-card-title>
          
          <v-divider></v-divider>
          
          <v-card-text class="pt-4 pb-4">
            <p class="text-body-1">
              Bạn có chắc chắn muốn xóa bạn đọc <strong>{{ reader.HoTen }}</strong>?
            </p>
            <p class="text-body-2 mt-2 text-red">
              Hành động này không thể hoàn tác.
            </p>
          </v-card-text>
          
          <v-divider></v-divider>
          
          <v-card-actions class="justify-center pa-4">
            <v-btn
              color="green"
              variant="flat"
              min-width="80"
              @click="deleteReader"
              :loading="deleteDialog.loading"
            >
              Xác nhận
            </v-btn>
            <v-btn
              color="grey"
              variant="flat"
              min-width="80"
              class="ml-4"
              @click="deleteDialog.show = false"
            >
              Hủy
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      
      <!-- Snackbar for notifications -->
      <v-snackbar
        v-model="snackbar.show"
        :color="snackbar.color"
        :timeout="3000"
      >
        {{ snackbar.text }}
        <template v-slot:actions>
          <v-btn variant="text" @click="snackbar.show = false">Đóng</v-btn>
        </template>
      </v-snackbar>
    </v-container>
  </div>
</template>

<script>
export default {
  name: "ReaderDetailPage",
  data() {
    return {
      reader: {},
      loading: true,
      error: null,
      deleteDialog: {
        show: false,
        loading: false
      },
      snackbar: {
        show: false,
        text: "",
        color: "success"
      }
    };
  },
  computed: {
    readerId() {
      return this.$route.params.id;
    }
  },
  mounted() {
    this.fetchReaderDetails();
  },
  methods: {
    async fetchReaderDetails() {
      this.loading = true;
      this.error = null;

      try {
        // Fetch reader data from API
        const response = await fetch(`https://26.193.242.15:8080/bandoc/get_user?user_id=${this.readerId}`);
        
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.detail || "Không thể tải thông tin bạn đọc");
        }
        
        const data = await response.json();
        this.reader = data;
      } catch (error) {
        console.error("Error fetching reader details:", error);
        this.error = `Lỗi: ${error.detail || "Không thể tải thông tin bạn đọc"}`;
      } finally {
        this.loading = false;
      }
    },
    
    formatDate(dateString) {
      if (!dateString) return "N/A";
      
      try {
        const date = new Date(dateString);
        return date.toLocaleDateString("vi-VN", {
          day: "2-digit",
          month: "2-digit",
          year: "numeric"
        });
      } catch (error) {
        return dateString;
      }
    },
    
    confirmDelete() {
      this.deleteDialog.show = true;
    },
    
    async deleteReader() {
      this.deleteDialog.loading = true;
      
      try {
        // Call the API to delete the reader
        const response = await fetch(`https://26.193.242.15:8080/bandoc/delete_user?user_id=${this.readerId}`, {
          method: 'DELETE',
          headers: {
            'Accept': 'application/json'
          }
        });
        
        const data = await response.json();
        
        // Handle potential error responses
        if (!response.ok) {
          let errorMsg = "Không thể xóa bạn đọc";
          if (response.status === 404) {
            errorMsg = data.detail || "Không tìm thấy độc giả";
          } else if (data.detail) {
            errorMsg = data.detail;
          }
          throw new Error(errorMsg);
        }
        
        // Show success detail
        this.snackbar = {
          show: true,
          text: `Đã xóa bạn đọc "${this.reader.HoTen}" thành công`,
          color: "success"
        };
        
        // Close dialog
        this.deleteDialog.show = false;
        
        // Redirect back to readers list after a delay
        setTimeout(() => {
          this.$router.push("/admin/readers");
        }, 1000);
        
      } catch (error) {
        console.error("Error deleting reader:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.detail || "Không thể xóa bạn đọc"}`,
          color: "error"
        };
        this.deleteDialog.show = false;
      } finally {
        this.deleteDialog.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.reader-detail-page {
  padding-bottom: 60px;
}

.info-label {
  font-weight: 500;
  width: 120px;
  color: rgba(0, 0, 0, 0.6);
}

.info-value {
  flex: 1;
}

.stat-box {
  border: 1px solid rgba(0, 0, 0, 0.12);
  border-radius: 4px;
}

.stat-value {
  font-size: 24px;
  font-weight: 500;
  margin-bottom: 8px;
}

.stat-label {
  color: rgba(0, 0, 0, 0.6);
}

@media (max-width: 600px) {
  .info-label {
    width: 100px;
  }
}
</style>