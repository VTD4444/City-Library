<template>
  <div class="book-detail-page">
    <v-container>
      <!-- Page title and back button -->
      <div class="d-flex align-center mb-6">
        <v-btn
          prepend-icon="mdi-arrow-left"
          variant="text"
          @click="$router.back()"
          class="mr-4"
        >
          Quay lại
        </v-btn>
      </div>

      <h1 class="text-h3 font-weight-bold mb-6 text-center">CHI TIẾT SÁCH</h1>

      <!-- Loading state -->
      <v-row v-if="loading" justify="center" class="my-10">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
        <div class="mt-2">Đang tải thông tin sách...</div>
      </v-row>

      <!-- Error state -->
      <v-alert v-else-if="error" type="error" class="mb-6">
        {{ error }}
      </v-alert>

      <!-- Book details form -->
      <v-form v-else ref="bookForm" @submit.prevent="saveBook">
        <v-row class="mt-6">
          <!-- Left column: Book image -->
          <v-col cols="12" md="4" class="d-flex justify-start">
            <v-card elevation="0" width="300">
              <div class="book-image-container mb-4">
                <img
                  v-if="imagePreview || book.AnhMinhHoaURL"
                  :src="imagePreview || book.AnhMinhHoaURL"
                  alt="Book cover"
                  class="book-image"
                />
                <v-icon v-else size="120" color="grey-lighten-1"
                  >mdi-book-variant</v-icon
                >
              </div>

              <v-card-text v-if="isEditMode">
                <v-file-input
                  v-model="imageFile"
                  accept="image/*"
                  prepend-icon=""
                  prepend-inner-icon="mdi-camera"
                  label="Tải ảnh lên"
                  hide-details
                  variant="outlined"
                  density="compact"
                  @change="previewImage"
                ></v-file-input>
              </v-card-text>
            </v-card>
          </v-col>

          <!-- Right column: Book details -->
          <v-col cols="12" md="8">
            <v-text-field
              v-model="book.TieuDe"
              label="Tiêu đề *"
              variant="outlined"
              density="comfortable"
              :readonly="!isEditMode"
              :rules="[(v) => !!v || 'Tiêu đề là bắt buộc']"
              class="text-h4 mb-2"
            ></v-text-field>
            <v-divider class="my-4"></v-divider>

            <div class="book-details">
              <v-row>
                <v-col cols="12" md="6">
                  <div class="detail-item">
                    <v-text-field
                      v-model="book.TacGia"
                      label="Tác giả *"
                      variant="outlined"
                      density="comfortable"
                      :readonly="!isEditMode"
                      :rules="[(v) => !!v || 'Tác giả là bắt buộc']"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.NamXuatBan"
                      label="Năm xuất bản"
                      variant="outlined"
                      density="comfortable"
                      type="number"
                      :readonly="!isEditMode"
                      :rules="[
                        (v) =>
                          !v ||
                          /^\d{4}$/.test(v) ||
                          'Năm xuất bản không hợp lệ',
                      ]"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.NXB"
                      label="Nhà xuất bản"
                      variant="outlined"
                      density="comfortable"
                      :readonly="!isEditMode"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.TheLoai"
                      label="Thể loại"
                      variant="outlined"
                      density="comfortable"
                      :readonly="!isEditMode"
                    ></v-text-field>
                  </div>
                </v-col>

                <v-col cols="12" md="6">
                  <div class="detail-item">
                    <v-text-field
                      v-model="book.SoTrang"
                      label="Số trang"
                      variant="outlined"
                      density="comfortable"
                      type="number"
                      :readonly="!isEditMode"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.MaSach"
                      label="Mã sách"
                      variant="outlined"
                      density="comfortable"
                      readonly
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.MaISBN"
                      label="Mã ISBN"
                      variant="outlined"
                      density="comfortable"
                      :readonly="!isEditMode"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.SoLuongHienCo"
                      label="Số lượng *"
                      variant="outlined"
                      density="comfortable"
                      type="number"
                      :readonly="!isEditMode"
                      :rules="[(v) => !!v || 'Số lượng là bắt buộc']"
                    ></v-text-field>
                  </div>
                </v-col>
              </v-row>
            </div>
          </v-col>
        </v-row>

        <!-- Book description section -->
        <v-row class="mt-8">
          <v-col cols="12">
            <h2 class="text-h5 mb-4 font-weight-bold">Mô tả sách</h2>
            <v-textarea
              v-model="book.MoTa"
              label="Mô tả"
              variant="outlined"
              rows="5"
              :readonly="!isEditMode"
              auto-grow
              density="comfortable"
              class="book-description"
            ></v-textarea>
          </v-col>
        </v-row>

        <!-- Book statistics (only in view mode) -->
        <v-row class="mt-8">
          <v-col cols="12" md="6">
            <h2 class="text-h5 mb-4 font-weight-bold">Thống kê sách</h2>
            <div class="d-flex flex-wrap">
              <div class="stat-item">
                <div class="stat-value">{{ book.SoLuongTong || 0 }}</div>
                <div class="stat-label">Số lượng tổng</div>
              </div>
              <div class="stat-item">
                <div class="stat-value">{{ book.SoLuongHienCo || 0 }}</div>
                <div class="stat-label">Số lượng còn lại</div>
              </div>
              <div class="stat-item">
                <div class="stat-value">{{ book.SoLuotDaMuon || 0 }}</div>
                <div class="stat-label">Lượt mượn</div>
              </div>
            </div>
          </v-col>
          <v-col cols="12" md="6">
            <!-- Action buttons -->
            <div class="mt-6 d-flex gap-4 justify-end">
              <v-btn
                v-if="!isEditMode"
                color="blue"
                size="large"
                @click="isEditMode = true"
                class="px-6"
                prepend-icon="mdi-pencil"
              >
                Chỉnh sửa
              </v-btn>
              <v-btn
                v-if="isEditMode"
                color="green"
                size="large"
                type="submit"
                class="mx-5 text-white"
                prepend-icon="mdi-check"
              >
                Lưu
              </v-btn>
              <v-btn
                v-if="isEditMode"
                variant="outlined"
                color="grey"
                size="large"
                class="px-6"
                @click="cancelEdit"
                prepend-icon="mdi-close"
              >
                Hủy
              </v-btn>
            </div>
          </v-col>
        </v-row>
      </v-form>

      <!-- Snackbar for notifications -->
      <v-snackbar
        v-model="snackbar.show"
        :color="snackbar.color"
        :timeout="3000"
      >
        {{ snackbar.text }}
        <template v-slot:actions>
          <v-btn variant="text" @click="snackbar.show = false"> Đóng </v-btn>
        </template>
      </v-snackbar>
    </v-container>
  </div>
</template>

<script>
export default {
  name: "BookDetailPage",
  data() {
    return {
      loading: true,
      error: null,
      book: {
        MaSach: "",
        TieuDe: "",
        TacGia: "",
        TheLoai: "",
        NXB: "",
        MaISBN: "",
        NamXuatBan: null,
        SoTrang: null,
        MoTa: "",
        SoLuongTong: null,
        SoLuongHienCo: null,
        SoLuotDaMuon: null,
        HinhAnh: "",
      },
      originalBook: {}, // To revert changes when cancel edit
      isEditMode: false,
      imageFile: null,
      imagePreview: null,
      borrowHistory: [],
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },
    };
  },
  computed: {
    bookId() {
      return this.$route.params.id;
    },
  },
  async mounted() {
    await this.fetchBookDetails();
  },
  methods: {
    async fetchBookDetails() {
      this.loading = true;
      this.error = null;

      try {
        const response = await fetch(
          `https://26.193.242.15:8080/books/${this.bookId}`,
          {
            method: "GET",
            headers: {
              Accept: "application/json",
            },
          }
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(errorData.message || "Không thể tải thông tin sách");
        }

        const data = await response.json();
        this.book = { ...data };
        this.originalBook = { ...data }; // Store original data for cancel functionality
      } catch (error) {
        console.error("Error fetching book details:", error);
        this.error = `Lỗi: ${error.message || "Không thể tải thông tin sách"}`;
      } finally {
        this.loading = false;
      }
    },

    previewImage(file) {
      if (!file) {
        this.imagePreview = null;
        return;
      }

      this.imagePreview = URL.createObjectURL(file);
    },

    cancelEdit() {
      this.isEditMode = false;
      this.book = { ...this.originalBook };
      this.imageFile = null;
      this.imagePreview = null;
    },

    async saveBook() {
      if (!this.$refs.bookForm.validate()) return;

      this.loading = true;
      try {
        // Sử dụng JSON thay vì FormData
        const bookData = {
          TieuDe: this.book.TieuDe || "",
          MaISBN: this.book.MaISBN || "",
          MaNXB: this.book.MaNXB || 0,
          MaTheLoai: this.book.MaTheLoai || 0,
          NamXuatBan: this.book.NamXuatBan || 0,
          SoTrang: this.book.SoTrang || 0,
          MoTa: this.book.MoTa || "",
          AnhMinhHoaURL: this.book.HinhAnh || "",
          SoLuongTong: this.book.SoLuongTong || 0,
        };

        // Update book
        const response = await fetch(
          `https://26.193.242.15:8080/books/update/${this.bookId}`,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
              Accept: "application/json",
            },
            body: JSON.stringify(bookData),
          }
        );

        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(
            errorData.message || "Không thể cập nhật thông tin sách"
          );
        }

        // Nếu cần upload hình ảnh riêng
        if (this.imageFile) {
          const imageData = new FormData();
          imageData.append("file", this.imageFile);

          const imageResponse = await fetch(
            `https://26.193.242.15:8080/books/${this.bookId}/upload-image`,
            {
              method: "POST",
              body: imageData,
            }
          );

          if (!imageResponse.ok) {
            console.error("Failed to upload image");
          }
        }

        // Update original book data
        const updatedData = await response.json();
        this.book = { ...updatedData };
        this.originalBook = { ...updatedData };

        // Reset edit mode
        this.isEditMode = false;
        this.imageFile = null;
        this.imagePreview = null;

        // Show success notification
        this.snackbar = {
          show: true,
          text: "Cập nhật sách thành công!",
          color: "success",
        };
      } catch (error) {
        console.error("Error updating book:", error);
        this.snackbar = {
          show: true,
          text: `Lỗi: ${error.message || "Không thể cập nhật thông tin sách"}`,
          color: "error",
        };
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.book-detail-page {
  padding-bottom: 40px;
}

.book-image-container {
  width: 100%;
  height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background-color: #f5f5f5;
  border-radius: 8px;
  border: 1px dashed rgba(0, 0, 0, 0.12);
}

.book-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.stat-item {
  text-align: center;
  margin-right: 32px;
  margin-bottom: 16px;
}

.stat-value {
  font-size: 24px;
  font-weight: 600;
  color: #2e7d32;
}

.stat-label {
  font-size: 14px;
  color: rgba(0, 0, 0, 0.6);
}

/* Read-only fields styling */
:deep(.v-field--variant-outlined.v-field--disabled) {
  opacity: 0.8;
}
</style>
