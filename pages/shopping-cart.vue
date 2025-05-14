<template>
  <div class="shopping-cart-page">
    <v-container>
      <h1 class="text-h4 text-center font-weight-bold mb-6">GIỎ SÁCH</h1>

      <!-- Hiển thị khi giỏ hàng trống -->
      <v-card v-if="!cartItems.length" elevation="0" class="pa-6 text-center">
        <v-icon size="64" color="grey" class="mb-4">mdi-cart-outline</v-icon>
        <h2 class="text-h5 mb-2">Giỏ sách của bạn đang trống</h2>
        <p class="mb-6 text-body-1">
          Thêm sách vào giỏ để có thể mượn nhiều đầu sách cùng lúc
        </p>
        <v-btn color="primary" elevation="0" rounded @click="$router.push('/books')">
          <v-icon start>mdi-book-open-variant</v-icon>
          Tiếp tục tìm sách
        </v-btn>
      </v-card>

      <!-- Hiển thị khi có sách trong giỏ -->
      <template v-else>
        <!-- Danh sách sách -->
        <v-card class="mb-6" elevation="0">
          <v-data-table
            v-model="selectedItems"
            :headers="tableHeaders"
            :items="cartItemsWithIndex"
            item-value="index"
            show-select
            class="cart-table"
            hide-default-footer
          >
            <!-- Cột STT -->
            <template v-slot:item.stt="{ item }">
              {{ item.stt }}
            </template>

            <!-- Cột mã sách -->
            <template v-slot:item.MaSach="{ item }">
              {{ item.MaSach }}
            </template>

            <!-- Cột tên sách -->
            <template v-slot:item.TieuDe="{ item }">
              {{ item.TieuDe }}
            </template>

            <!-- Cột xem chi tiết -->
            <template v-slot:item.actions="{ item }">
              <v-btn
                variant="text"
                color="primary"
                size="small"
                @click="viewBookDetail(item.MaSach)"
              >
                Xem
              </v-btn>
            </template>
          </v-data-table>
        </v-card>

        <!-- Phần bên dưới bảng -->
        <div class="d-flex justify-space-between align-center mb-5">
          <!-- Nút tiếp tục tìm sách và xóa giỏ sách -->
          <div>
            <v-btn
              color="primary"
              variant="text"
              class="mr-3"
              @click="$router.push('/books')"
            >
              <v-icon start>mdi-arrow-left</v-icon>
              Tiếp tục tìm sách
            </v-btn>
            <v-btn
              color="error"
              variant="text"
              @click="showClearCartDialog = true"
            >
              <v-icon start>mdi-delete-sweep</v-icon>
              Xóa giỏ sách
            </v-btn>
          </div>

          <!-- Tổng số lượng -->
          <div class="d-flex align-center">
            <span class="text-h6 mr-2">Tổng:</span>
            <v-text-field
              v-model="selectedItems.length"
              density="compact"
              variant="outlined"
              readonly
              hide-details
              class="total-count-field"
              style="width: 60px"
            ></v-text-field>
          </div>
        </div>

        <!-- Nút mượn sách (căn giữa, dưới cùng) -->
        <div class="d-flex justify-center mt-8">
          <v-btn
            color="green-darken-1"
            size="large"
            min-width="180"
            elevation="0"
            @click="proceedToCheckout"
            :disabled="!selectedItems.length"
          >
            Mượn sách
          </v-btn>
        </div>
      </template>
    </v-container>

    <!-- Dialog xác nhận xóa giỏ hàng -->
    <v-dialog v-model="showClearCartDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h5">Xóa giỏ sách?</v-card-title>
        <v-card-text>
          Bạn có chắc chắn muốn xóa tất cả sách trong giỏ không? Hành động này
          không thể hoàn tác.
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="grey"
            variant="text"
            @click="showClearCartDialog = false"
          >
            Hủy
          </v-btn>
          <v-btn color="error" @click="clearCart"> Xóa giỏ sách </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Snackbar thông báo -->
    <v-snackbar v-model="snackbar.show" :color="snackbar.color" :timeout="3000">
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn variant="text" @click="snackbar.show = false">Đóng</v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>
export default {
  name: "ShoppingCart",
  data() {
    return {
      cartItems: [],
      selectedItems: [], // Danh sách các sách được chọn để mượn
      tableHeaders: [
        { title: "STT", key: "stt", align: "center", width: "80px" },
        { title: "Mã sách", key: "MaSach", align: "center", width: "150px" },
        { title: "Tên sách", key: "TieuDe" },
        {
          title: "",
          key: "actions",
          sortable: false,
          align: "center",
          width: "100px",
        },
      ],
      showClearCartDialog: false,
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },
    };
  },
  computed: {
    // Thêm STT vào mỗi item để hiển thị
    cartItemsWithIndex() {
      return this.cartItems.map((item, index) => ({
        ...item,
        stt: index + 1,
        index: index, // Dùng để xác định item khi chọn
      }));
    },
  },
  mounted() {
    this.loadCartItems();
  },
  methods: {
    loadCartItems() {
      // Kiểm tra đăng nhập
      const isLoggedIn = !!localStorage.getItem("userLogin");
      if (!isLoggedIn) {
        this.$router.push("/login");
        return;
      }

      // Lấy giỏ hàng từ localStorage
      const cart = JSON.parse(localStorage.getItem("cart") || "[]");
      this.cartItems = cart;

      // Mặc định chọn tất cả sách
      this.selectedItems = this.cartItems.map((_, index) => index);
    },

    removeItem(index) {
      // Xóa sách khỏi giỏ hàng
      this.cartItems.splice(index, 1);
      this.saveCart();

      // Cập nhật danh sách đã chọn
      this.selectedItems = this.selectedItems
        .filter((itemIndex) => itemIndex < index)
        .concat(
          this.selectedItems
            .filter((itemIndex) => itemIndex > index)
            .map((itemIndex) => itemIndex - 1)
        );

      this.snackbar.text = "Đã xóa sách khỏi giỏ hàng";
      this.snackbar.color = "success";
      this.snackbar.show = true;
    },

    clearCart() {
      // Xóa toàn bộ giỏ hàng
      this.cartItems = [];
      this.selectedItems = [];
      this.saveCart();
      this.showClearCartDialog = false;

      this.snackbar.text = "Đã xóa toàn bộ giỏ hàng";
      this.snackbar.color = "success";
      this.snackbar.show = true;
    },

    saveCart() {
      // Lưu giỏ hàng vào localStorage
      localStorage.setItem("cart", JSON.stringify(this.cartItems));
    },

    viewBookDetail(bookId) {
      // Chuyển hướng đến trang chi tiết sách
      this.$router.push(`/books/${bookId}`);
    },

    async proceedToCheckout() {
      // Lấy ra danh sách các sách được chọn để mượn
      const selectedBooks = this.selectedItems.map(
        (index) => this.cartItems[index]
      );

      if (!selectedBooks.length) {
        this.snackbar.text = "Vui lòng chọn sách để mượn";
        this.snackbar.color = "warning";
        this.snackbar.show = true;
        return;
      }

      try {
        // Hiển thị thông báo đang xử lý
        this.snackbar.text = "Đang xử lý yêu cầu mượn sách...";
        this.snackbar.color = "info";
        this.snackbar.show = true;

        // Lấy thông tin người dùng từ localStorage
        const userLogin = JSON.parse(localStorage.getItem("userLogin"));
        if (!userLogin || !userLogin.MaDocGia) {
          throw new Error(
            "Không tìm thấy thông tin độc giả, vui lòng đăng nhập lại"
          );
        }

        // Tạo dữ liệu gửi lên API
        const requestData = {
          MaDocGia: userLogin.MaDocGia,
          GhiChu: "",
          ChiTietPhieuMuons: selectedBooks.map((book) => ({
            MaSach: book.MaSach,
          })),
        };

        // Gọi API tạo phiếu mượn
        const response = await fetch(
          "http://26.193.242.15:8000/phieumuons/create",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(requestData),
          }
        );

        const data = await response.json();

        // Xử lý phản hồi từ API
        if (!response.ok) {
          throw new Error(
            data.message || "Không thể tạo phiếu mượn, vui lòng thử lại sau"
          );
        }

        // Lưu thông báo vào localStorage để hiển thị trong notification panel
        if (data.thong_bao) {
          // Lấy thông báo hiện tại từ localStorage hoặc khởi tạo mảng mới
          const existingNotifications = JSON.parse(
            localStorage.getItem("notifications") || "[]"
          );

          // Thêm thông báo mới vào đầu mảng
          existingNotifications.unshift({
            id: data.thong_bao.MaThongBao,
            type: data.thong_bao.LoaiThongBao,
            message: data.thong_bao.NoiDung,
            date: data.thong_bao.NgayTao,
            status: data.thong_bao.TrangThai,
            data: {
              MaPhieuMuon: data.thong_bao.MaPhieuMuon,
            },
          });

          // Lưu lại vào localStorage
          localStorage.setItem(
            "notifications",
            JSON.stringify(existingNotifications)
          );

          // Kích hoạt sự kiện để header cập nhật số lượng thông báo
          this.$root.$emit("new-notification");
        }

        // Xóa sách đã mượn khỏi giỏ hàng
        this.selectedItems.sort((a, b) => b - a); // Sắp xếp giảm dần để xóa từ cuối lên
        this.selectedItems.forEach((index) => {
          this.cartItems.splice(index, 1);
        });

        // Lưu giỏ hàng sau khi đã xóa sách đã mượn
        this.saveCart();

        // Reset danh sách đã chọn
        this.selectedItems = this.cartItems.map((_, index) => index);

        // Hiển thị thông báo thành công
        this.snackbar.text = data.message || "Mượn sách thành công!";
        this.snackbar.color = "success";
        this.snackbar.show = true;

        // Chuyển hướng đến trang phiếu mượn sau 1.5 giây
        setTimeout(() => {
          this.$router.push("/user/borrowed-tickets/");
        }, 1000);
      } catch (error) {
        console.error("Error creating loan:", error);

        // Hiển thị thông báo lỗi
        this.snackbar.text = error.message || "Đã xảy ra lỗi khi mượn sách";
        this.snackbar.color = "error";
        this.snackbar.show = true;
      }
    },
  },
};
</script>

<style scoped>
.shopping-cart-page {
  padding-bottom: 40px;
}

.cart-table :deep(th) {
  font-weight: 600 !important;
  color: rgba(0, 0, 0, 0.8) !important;
  background-color: #e8f5e9 !important;
}

.cart-table :deep(td) {
  height: 60px;
}

/* Tạo border cho bảng */
.cart-table :deep(table) {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.cart-table :deep(th),
.cart-table :deep(td) {
  border-right: 1px solid rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

/* Style cho ô tổng số lượng */
.total-count-field :deep(input) {
  text-align: center;
  font-weight: 500;
  font-size: 16px;
}
</style>
