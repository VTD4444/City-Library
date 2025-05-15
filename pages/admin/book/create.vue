<template>
  <div class="book-create-page">
    <v-container>
      <h1 class="text-h3 font-weight-bold mb-6 text-center">THÊM SÁCH MỚI</h1>

      <!-- Loading state (if needed) -->
      <v-row v-if="loading" justify="center" class="my-10">
        <v-progress-circular
          indeterminate
          color="green"
          size="64"
        ></v-progress-circular>
      </v-row>

      <v-alert v-if="error" type="error" class="mt-4" closable>{{
        error
      }}</v-alert>

      <!-- Form content -->
      <v-form v-model="isFormValid" @submit.prevent="submitBook">
        <v-row class="mt-6">
          <!-- Left column - Book image preview -->
          <v-col cols="12" md="4" class="d-flex justify-start">
            <v-card elevation="0" width="300">
              <v-img
                :src="
                  previewImage ||
                  'https://www.svgrepo.com/show/508699/landscape-placeholder.svg'
                "
                alt="Book cover preview"
                class="book-cover"
                height="400"
                cover
              ></v-img>

              <v-card-text>
                <v-text-field
                  v-model="book.AnhMinhHoaURL"
                  label="URL hình ảnh"
                  variant="outlined"
                  hint="Nhập đường dẫn đến hình ảnh bìa sách"
                  placeholder="https://example.com/book-cover.jpg"
                  @input="updatePreview"
                ></v-text-field>
              </v-card-text>
            </v-card>
          </v-col>

          <!-- Right column - Book details -->
          <v-col cols="12" md="8">
            <v-text-field
              v-model="book.TieuDe"
              label="Tiêu đề sách *"
              variant="outlined"
              :rules="[rules.required]"
              class="text-h4 mb-2"
              hide-details
            ></v-text-field>
            <v-divider class="my-4"></v-divider>

            <div class="book-details">
              <v-row>
                <v-col cols="12" md="6">
                  <div class="detail-item">
                    <v-text-field
                      v-model="book.TenTacGia"
                      label="Tác giả *"
                      variant="outlined"
                      :rules="[rules.required]"
                      density="compact"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.NamXuatBan"
                      label="Năm xuất bản *"
                      variant="outlined"
                      type="number"
                      min="1800"
                      :max="new Date().getFullYear()"
                      :rules="[rules.required, rules.year]"
                      density="compact"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.NXB"
                      label="Nhà xuất bản *"
                      variant="outlined"
                      :rules="[rules.required]"
                      density="compact"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.TheLoai"
                      label="Thể loại *"
                      variant="outlined"
                      :rules="[rules.required]"
                      density="compact"
                    ></v-text-field>
                  </div>
                </v-col>

                <v-col cols="12" md="6">
                  <div class="detail-item">
                    <v-text-field
                      v-model="book.SoTrang"
                      label="Số trang *"
                      variant="outlined"
                      type="number"
                      min="1"
                      :rules="[rules.required, rules.positive]"
                      density="compact"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.MaISBN"
                      label="Mã ISBN *"
                      variant="outlined"
                      :rules="[rules.required]"
                      density="compact"
                    ></v-text-field>
                  </div>

                  <div class="detail-item">
                    <v-text-field
                      v-model="book.SoLuongTong"
                      label="Số lượng *"
                      variant="outlined"
                      type="number"
                      min="0"
                      :rules="[rules.required, rules.nonNegative]"
                      density="compact"
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
              label="Mô tả chi tiết *"
              variant="outlined"
              rows="6"
              :rules="[rules.required]"
              hint="Nhập mô tả chi tiết về nội dung sách"
              class="book-description"
            ></v-textarea>
          </v-col>
        </v-row>
        <!-- Action buttons -->
        <div class="mt-6 d-flex gap-4 justify-end">
          <v-btn
            color="green-darken-1"
            size="large"
            class="px-6 text-white"
            type="submit"
            :loading="loading"
            :disabled="!isFormValid"
          >
            <v-icon left class="mr-2">mdi-plus</v-icon>
            Tạo sách
          </v-btn>

          <v-btn
            variant="outlined"
            color="grey"
            size="large"
            class="mx-5"
            :to="'/admin/manage-books'"
          >
            <v-icon left class="mr-2">mdi-cancel</v-icon>
            Hủy
          </v-btn>
        </div>
      </v-form>
    </v-container>

    <!-- Snackbar thông báo -->
    <v-snackbar v-model="snackbar.show" :color="snackbar.color" :timeout="3000">
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="snackbar.show = false"> Đóng </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>
export default {
  name: "BookCreatePage",
  data() {
    return {
      isFormValid: false,
      loading: false,
      previewImage: null,
      newCategory: "",
      book: {
        TieuDe: "",
        MaISBN: "",
        TenTacGia: "",
        NXB: "",
        TheLoai: "",
        NamXuatBan: new Date().getFullYear(),
        SoTrang: "",
        MoTa: "",
        AnhMinhHoaURL: "",
        SoLuongTong: 1,
      },
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },
      rules: {
        required: (value) => !!value || "Thông tin này là bắt buộc",
        positive: (value) => value > 0 || "Giá trị phải lớn hơn 0",
        nonNegative: (value) => value >= 0 || "Giá trị không được âm",
        year: (value) => {
          const currentYear = new Date().getFullYear();
          return (
            (value >= 1800 && value <= currentYear) ||
            `Năm phải từ 1800 đến ${currentYear}`
          );
        },
      },
    };
  },
  methods: {
    updatePreview() {
      // Cập nhật preview khi URL hình ảnh thay đổi
      this.previewImage = this.book.AnhMinhHoaURL || null;
    },

    async submitBook() {
      if (!this.isFormValid) return;

      this.loading = true;

      try {
        // Adjust TheLoai handling for the object format
        const theLoai = typeof this.book.TheLoai === 'object' 
          ? this.book.TheLoai.TenTheLoai 
          : this.book.TheLoai;
        
        // Chuẩn bị dữ liệu gửi đi
        const bookData = {
          TieuDe: this.book.TieuDe,
          MaISBN: this.book.MaISBN,
          TenTacGia: this.book.TenTacGia,
          NXB: this.book.NXB,
          TheLoai: theLoai,
          NamXuatBan: parseInt(this.book.NamXuatBan),
          SoTrang: parseInt(this.book.SoTrang),
          MoTa: this.book.MoTa,
          AnhMinhHoaURL: this.book.AnhMinhHoaURL || null,
          SoLuongTong: parseInt(this.book.SoLuongTong),
        };

        // Gọi API để tạo sách mới
        const response = await fetch("https://26.193.242.15:8080/books/create", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(bookData),
        });

        const data = await response.json();

        if (!response.ok) {
          throw new Error(data.message || "Không thể tạo sách mới");
        }

        // Hiển thị thông báo thành công
        this.snackbar.color = "success";
        this.snackbar.text = "Tạo sách mới thành công!";
        this.snackbar.show = true;

        // Chuyển hướng về trang danh sách sách sau 1.5 giây
        setTimeout(() => {
          this.$router.push("/admin/manage-books");
        }, 1500);
      } catch (error) {
        console.error("Error creating book:", error);

        // Hiển thị thông báo lỗi
        this.snackbar.color = "error";
        this.snackbar.text = error.message || "Đã xảy ra lỗi khi tạo sách mới";
        this.snackbar.show = true;
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.book-create-page {
  padding-bottom: 50px;
}

.book-cover {
  object-fit: contain;
}

.detail-item {
  margin-bottom: 16px;
}

.book-description {
  line-height: 1.8;
}

:deep(.v-text-field .v-field__input) {
  padding-top: 8px;
  padding-bottom: 8px;
}

:deep(.v-text-field .v-field--variant-outlined .v-field__outline) {
  color: rgba(0, 0, 0, 0.12);
}

:deep(.v-text-field .v-field:hover .v-field__outline) {
  color: rgba(0, 0, 0, 0.24);
}

:deep(.v-label) {
  color: #686868 !important;
  font-weight: 500 !important;
  opacity: 0.75 !important;
}
</style>
