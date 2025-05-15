<template>
  <div class="book-detail-page">
    <v-container>
      <!-- Loading và error states -->
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

      <!-- Nội dung chi tiết sách -->
      <template v-if="book && !loading">
        <v-row class="mt-6">
          <!-- Hình ảnh sách -->
          <v-col cols="12" md="4" class="d-flex justify-start">
            <v-card class="book-image-container elevation-3" width="300">
              <v-img
                :src="getBookImage()"
                :alt="book.TieuDe"
                class="book-cover"
                height="400"
                cover
              ></v-img>
            </v-card>
          </v-col>

          <!-- Thông tin chi tiết sách -->
          <v-col cols="12" md="8">
            <h1 class="text-h3 font-weight-bold mb-2">{{ book.TieuDe }}</h1>
            <v-divider class="my-4"></v-divider>

            <div class="book-details">
              <v-row>
                <v-col cols="12" md="6">
                  <div class="detail-item">
                    <span class="detail-label">Tác giả:</span>
                    <span class="detail-value">{{ book.TacGia }}</span>
                  </div>

                  <div class="detail-item">
                    <span class="detail-label">Năm xuất bản:</span>
                    <span class="detail-value">{{ book.NamXuatBan }}</span>
                  </div>

                  <div class="detail-item">
                    <span class="detail-label">Nhà xuất bản:</span>
                    <span class="detail-value">{{ book.NXB }}</span>
                  </div>

                  <div class="detail-item">
                    <span class="detail-label">Thể loại:</span>
                    <span class="detail-value">{{ book.TheLoai }}</span>
                  </div>
                </v-col>

                <v-col cols="12" md="6">
                  <div class="detail-item">
                    <span class="detail-label">Số trang:</span>
                    <span class="detail-value">{{ book.SoTrang }}</span>
                  </div>

                  <div class="detail-item">
                    <span class="detail-label">Mã sách:</span>
                    <span class="detail-value">{{ book.MaSach }}</span>
                  </div>

                  <div class="detail-item">
                    <span class="detail-label">Mã ISBN:</span>
                    <span class="detail-value">{{ book.MaISBN }}</span>
                  </div>

                  <div class="detail-item">
                    <span class="detail-label">Số lượng:</span>
                    <span class="detail-value"
                      >{{ book.SoLuongHienCo || 0 }} quyển</span
                    >
                  </div>
                </v-col>
              </v-row>

              <!-- Nút thao tác -->
              <div class="mt-6 d-flex gap-4">
                <v-btn
                  color="green-darken-1"
                  size="large"
                  class="px-6 text-white"
                  :disabled="book.SoLuongPhonMuon <= 0"
                  @click="borrowBook"
                >
                  <v-icon left class="mr-1">mdi-bookmark-check</v-icon>
                  Mượn sách
                </v-btn>

                <v-btn
                  variant="outlined"
                  color="green-darken-1"
                  size="large"
                  class="px-6 mx-4"
                  @click="addToCart"
                >
                  <v-icon left class="mr-1">mdi-plus</v-icon>
                  Thêm vào giỏ sách
                </v-btn>
              </div>
            </div>
          </v-col>
        </v-row>

        <!-- Mô tả sách -->
        <v-row class="mt-8">
          <v-col cols="12">
            <h2 class="text-h5 mb-4 font-weight-bold">Giới thiệu sách</h2>
            <div class="book-description text-body-1">
              {{ book.MoTa }}
            </div>
          </v-col>
        </v-row>

        <!-- Sách liên quan -->
        <!-- <v-row class="mt-8">
          <v-col cols="12">
            <h2 class="text-h5 mb-4 font-weight-bold">Sách cùng thể loại</h2>
            <v-alert v-if="true" type="info" variant="tonal" class="mb-0">
              Đang tải danh sách sách liên quan...
            </v-alert>
            Danh sách sách cùng thể loại sẽ hiển thị ở đây
          </v-col>
        </v-row> -->
      </template>
    </v-container>

    <!-- Add borrow confirmation dialog -->
    <v-dialog v-model="confirmDialog.show" max-width="500px" persistent>
      <v-card>
        <v-card-title class="notification-header d-flex align-center">
          <span>Thông báo:</span>
          <v-spacer></v-spacer>
          <v-btn elevation="0" icon @click="confirmDialog.show = false">
            <v-icon color="red">mdi-close</v-icon>
          </v-btn>
        </v-card-title>

        <v-divider></v-divider>

        <v-card-text class="pt-4 pb-4">
          <p class="text-body-1">
            Bạn có muốn mượn sách "{{ book.TieuDe }}" không?
          </p>
          <p class="text-body-2 mt-2">
            Sau khi xác nhận, thủ thư sẽ duyệt phiếu mượn của bạn.
          </p>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions class="justify-center pa-4">
          <v-btn
            color="success"
            min-width="80"
            @click="confirmBorrow"
            :loading="confirmDialog.loading"
          >
            Xác nhận
          </v-btn>
          <v-btn
            color="grey"
            variant="outlined"
            min-width="80"
            class="ml-4"
            @click="confirmDialog.show = false"
          >
            Hủy
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

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
  name: "BookDetailPage",
  data() {
    return {
      book: null,
      loading: true,
      error: null,
      bookId: null,
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },
      confirmDialog: {
        show: false,
        loading: false,
      },
    };
  },
  mounted() {
    // Lấy ID sách từ tham số URL
    this.bookId = this.$route.params.id;
    this.fetchBookDetails();
  },
  methods: {
    async fetchBookDetails() {
      this.loading = true;
      this.error = null;

      try {
        // Gọi API để lấy thông tin chi tiết sách
        const response = await fetch(
          `https://26.193.242.15:8080/books/${this.bookId}`
        );

        if (!response.ok) {
          throw new Error(
            "Không thể tải thông tin sách. Vui lòng thử lại sau."
          );
        }

        const data = await response.json();
        this.book = data;
      } catch (error) {
        console.error("Error fetching book details:", error);
        this.error = error.message || "Có lỗi xảy ra khi tải thông tin sách";
      } finally {
        this.loading = false;
      }
    },

    getBookImage() {
      // Xử lý đường dẫn hình ảnh
      if (!this.book || !this.book.AnhMinhHoaURL) {
        return "https://www.svgrepo.com/show/508699/landscape-placeholder.svg"; // Hình đại diện nếu không có ảnh
      }

      // Nếu đường dẫn là URL đầy đủ
      if (this.book.AnhMinhHoaURL.startsWith("http")) {
        return this.book.AnhMinhHoaURL;
      }

      // Nếu là đường dẫn tương đối, thêm base URL
      return `https://26.193.242.15:8080${this.book.AnhMinhHoaURL}`;
    },

    borrowBook() {
      // Kiểm tra đăng nhập
      const isLoggedIn = !!localStorage.getItem("userLogin");

      if (!isLoggedIn) {
        this.snackbar.color = "error";
        this.snackbar.text = "Vui lòng đăng nhập để mượn sách";
        this.snackbar.show = true;

        // Chuyển hướng đến trang đăng nhập sau 1.5 giây
        setTimeout(() => {
          this.$router.push("/login");
        }, 1500);
        return;
      }

      // Show confirmation dialog
      this.confirmDialog.show = true;
    },

    async confirmBorrow() {
      try {
        this.confirmDialog.loading = true;

        // Get user info
        const userLogin = JSON.parse(localStorage.getItem("userLogin"));
        if (!userLogin || !userLogin.MaDocGia) {
          throw new Error(
            "Không tìm thấy thông tin độc giả, vui lòng đăng nhập lại"
          );
        }

        // Prepare API request data
        const requestData = {
          MaDocGia: userLogin.MaDocGia,
          GhiChu: "",
          ChiTietPhieuMuons: [
            {
              MaSach: this.book.MaSach,
            },
          ],
        };

        // Call API to create borrow ticket
        const response = await fetch(
          "https://26.193.242.15:8080/phieumuons/create",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(requestData),
          }
        );

        const data = await response.json();

        // Handle API response
        if (!response.ok) {
          // Extract error message from different response formats
          let errorMessage = "Không thể tạo phiếu mượn, vui lòng thử lại sau.";

          if (data.detail) {
            // Handle specific error format with 'detail' field
            errorMessage = data.detail;
          } else if (data.message) {
            // Handle error format with 'message' field
            errorMessage = data.message;
          }

          throw new Error(errorMessage);
        }

        this.confirmDialog.show = false;

        // Show success message
        this.snackbar.color = "success";
        this.snackbar.text =
          "Tạo phiếu mượn thành công! Thủ thư sẽ xác nhận phiếu của bạn.";
        this.snackbar.show = true;
      } catch (error) {
        console.error("Error borrowing book:", error);

        // Close dialog
        this.confirmDialog.show = false;

        // Show error in snackbar
        this.snackbar.color = "error";
        this.snackbar.text = error.message || "Đã xảy ra lỗi khi mượn sách";
        this.snackbar.show = true;
      } finally {
        this.confirmDialog.loading = false;
      }
    },

    addToCart() {
      // Kiểm tra đăng nhập
      const isLoggedIn = !!localStorage.getItem("userLogin");
      if (!isLoggedIn) {
        this.snackbar.color = "error";
        this.snackbar.text = "Vui lòng đăng nhập để thêm sách vào giỏ";
        this.snackbar.show = true;

        setTimeout(() => {
          this.$router.push("/login");
        }, 1000);
        return;
      }

      // Lấy giỏ hàng từ localStorage
      let cart = JSON.parse(localStorage.getItem("cart") || "[]");

      // Kiểm tra sách đã có trong giỏ chưa
      const existingBookIndex = cart.findIndex(
        (item) => item.MaSach === this.book.MaSach
      );

      if (existingBookIndex >= 0) {
        // Sách đã có trong giỏ
        this.snackbar.color = "info";
        this.snackbar.text = "Sách này đã có trong giỏ của bạn";
        this.snackbar.show = true;
        return;
      }

      // Thêm sách vào giỏ hàng (mặc định số lượng = 1)
      cart.push({
        MaSach: this.book.MaSach,
        TieuDe: this.book.TieuDe,
        TacGia: this.book.TacGia,
        TheLoai: this.book.TheLoai,
        AnhMinhHoaURL: this.book.AnhMinhHoaURL,
        quantity: 1, // Mỗi loại sách chỉ được mượn 1 cuốn
      });

      // Cập nhật giỏ hàng vào localStorage
      localStorage.setItem("cart", JSON.stringify(cart));

      // Thông báo thành công
      this.snackbar.color = "success";
      this.snackbar.text = "Đã thêm sách vào giỏ";
      this.snackbar.show = true;
    },
  },
};
</script>

<style scoped>
.book-detail-page {
  padding-bottom: 50px;
}

.book-image-container {
  position: relative;
  overflow: hidden;
  transition: transform 0.3s;
}

.book-image-container:hover {
  transform: scale(1.02);
}

.book-cover {
  object-fit: contain;
}

.detail-item {
  margin-bottom: 16px;
  font-size: 16px;
}

.detail-label {
  font-weight: 600;
  min-width: 120px;
  display: inline-block;
  color: rgba(0, 0, 0, 0.7);
}

.detail-value {
  font-weight: 400;
}

.book-description {
  line-height: 1.8;
  text-align: justify;
}

/* Add this new style for the notification header */
.notification-header {
  background-color: #f8f8f8;
  color: rgba(0, 0, 0, 0.87);
  font-weight: 500;
  padding: 16px;
}
</style>
