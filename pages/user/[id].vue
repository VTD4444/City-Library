<template>
  <div class="user-profile-page">
    <v-container>
      <v-row>
        <v-col cols="12" md="3">
          <!-- Phần thẻ thư viện -->
          <v-card class="rounded-lg mb-6" elevation="0">
            <v-card-title class="text-h5 pb-2">
              Thẻ thư viện
              <div class="text-subtitle-2 text-grey font-weight-regular">
                Chào {{ userInfo.HoTen }}
              </div>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text class="d-flex align-center justify-center py-6">
              <v-avatar color="green" size="100">
                <v-icon size="64" color="white">mdi-account</v-icon>
              </v-avatar>
            </v-card-text>
          </v-card>

          <!-- Các liên kết khác -->
          <v-list rounded="lg" nav>
            <v-list-item
              to="/user/borrow-tickets"
              prepend-icon="mdi-book-open-variant"
              title="Lịch sử mượn sách"
            ></v-list-item>
            <v-list-item
              @click="logout"
              prepend-icon="mdi-logout"
              title="Đăng xuất"
            ></v-list-item>
          </v-list>
        </v-col>

        <v-col cols="12" md="9">
          <v-card class="rounded-lg" elevation="0">
            <v-card-title class="text-h5">
              Thông tin cá nhân
              <div class="text-subtitle-2 text-grey font-weight-regular">
                Chỉnh sửa thông tin cá nhân của bạn
              </div>
            </v-card-title>
            <v-divider></v-divider>

            <v-form ref="form" v-model="valid" @submit.prevent="saveUserInfo">
              <v-card-text class="pt-4">
                <v-row>
                  <!-- Tên đăng nhập -->
                  <v-col cols="12" md="6">
                    <div class="text-subtitle-2 mb-1">Tên đăng nhập:</div>
                    <v-text-field
                      v-model="userInfo.TenDangNhap"
                      variant="outlined"
                      density="comfortable"
                      readonly
                      disabled
                      hide-details
                      class="mb-3"
                    ></v-text-field>
                  </v-col>

                  <!-- Họ tên -->
                  <v-col cols="12" md="6">
                    <div class="text-subtitle-2 mb-1">Họ tên:</div>
                    <v-text-field
                      v-model="userInfo.HoTen"
                      variant="outlined"
                      density="comfortable"
                      :rules="[(v) => !!v || 'Vui lòng nhập họ tên']"
                      append-inner-icon="mdi-pencil"
                      hide-details
                      class="mb-3"
                    ></v-text-field>
                  </v-col>

                  <!-- Email -->
                  <v-col cols="12" md="6">
                    <div class="text-subtitle-2 mb-1">Email:</div>
                    <v-text-field
                      v-model="userInfo.Email"
                      variant="outlined"
                      density="comfortable"
                      :rules="emailRules"
                      append-inner-icon="mdi-pencil"
                      hide-details
                      class="mb-3"
                    ></v-text-field>
                  </v-col>

                  <!-- Số điện thoại -->
                  <v-col cols="12" md="6">
                    <div class="text-subtitle-2 mb-1">Số điện thoại:</div>
                    <v-text-field
                      v-model="userInfo.SoDienThoai"
                      variant="outlined"
                      density="comfortable"
                      :rules="phoneRules"
                      append-inner-icon="mdi-pencil"
                      hide-details
                      class="mb-3"
                    ></v-text-field>
                  </v-col>
                </v-row>

                <!-- Đổi mật khẩu -->
                <v-expansion-panels
                  variant="accordion"
                  class="mt-4"
                  elevation="0"
                >
                  <v-expansion-panel>
                    <v-expansion-panel-title>
                      <div class="d-flex align-center">
                        <v-icon class="mr-2">mdi-lock</v-icon>
                        Đổi mật khẩu
                      </div>
                    </v-expansion-panel-title>
                    <v-expansion-panel-text>
                      <v-row>
                        <!-- Mật khẩu hiện tại -->
                        <v-col cols="12" md="4">
                          <div class="text-subtitle-2 mb-1">
                            Mật khẩu hiện tại:
                          </div>
                          <v-text-field
                            v-model="passwordData.MatKhau"
                            variant="outlined"
                            density="comfortable"
                            :type="showPasswords.current ? 'text' : 'password'"
                            :append-inner-icon="
                              showPasswords.current ? 'mdi-eye-off' : 'mdi-eye'
                            "
                            @click:append-inner="
                              showPasswords.current = !showPasswords.current
                            "
                            hide-details
                            class="mb-3"
                          ></v-text-field>
                        </v-col>

                        <!-- Mật khẩu mới -->
                        <v-col cols="12" md="4">
                          <div class="text-subtitle-2 mb-1">Mật khẩu mới:</div>
                          <v-text-field
                            v-model="passwordData.NewPassword"
                            variant="outlined"
                            density="comfortable"
                            :type="showPasswords.new ? 'text' : 'password'"
                            :append-inner-icon="
                              showPasswords.new ? 'mdi-eye-off' : 'mdi-eye'
                            "
                            @click:append-inner="
                              showPasswords.new = !showPasswords.new
                            "
                            :rules="passwordRules"
                            hide-details
                            class="mb-3"
                          ></v-text-field>
                        </v-col>

                        <!-- Xác nhận mật khẩu mới -->
                        <v-col cols="12" md="4">
                          <div class="text-subtitle-2 mb-1">
                            Xác nhận mật khẩu mới:
                          </div>
                          <v-text-field
                            v-model="passwordData.ConfirmNewPassword"
                            variant="outlined"
                            density="comfortable"
                            :type="showPasswords.confirm ? 'text' : 'password'"
                            :append-inner-icon="
                              showPasswords.confirm ? 'mdi-eye-off' : 'mdi-eye'
                            "
                            @click:append-inner="
                              showPasswords.confirm = !showPasswords.confirm
                            "
                            :rules="[
                              (v) => !!v || 'Vui lòng nhập lại mật khẩu mới',
                              (v) =>
                                v === passwordData.NewPassword ||
                                'Mật khẩu không khớp',
                            ]"
                            hide-details
                            class="mb-3"
                          ></v-text-field>
                        </v-col>
                      </v-row>
                    </v-expansion-panel-text>
                  </v-expansion-panel>
                </v-expansion-panels>
              </v-card-text>

              <v-card-actions class="px-4 pb-4">
                <v-spacer></v-spacer>
                <v-btn
                  color="green-darken-1"
                  min-width="120"
                  type="submit"
                  :loading="loading"
                >
                  Lưu
                </v-btn>
              </v-card-actions>
            </v-form>
          </v-card>
        </v-col>
      </v-row>
    </v-container>

    <!-- Thông báo kết quả -->
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
  data() {
    return {
      valid: false,
      loading: false,
      userId: null,
      userInfo: {
        TenDangNhap: "",
        HoTen: "",
        Email: "",
        SoDienThoai: "",
      },
      passwordData: {
        MatKhau: "",
        NewPassword: "",
        ConfirmNewPassword: "",
      },
      showPasswords: {
        current: false,
        new: false,
        confirm: false,
      },
      snackbar: {
        show: false,
        text: "",
        color: "success",
      },
      emailRules: [
        (v) => !!v || "Vui lòng nhập email",
        (v) => /.+@.+\..+/.test(v) || "Email không hợp lệ",
      ],
      phoneRules: [
        (v) => !v || /^[0-9]{10,11}$/.test(v) || "Số điện thoại không hợp lệ",
      ],
      passwordRules: [
        (v) => !v || v.length >= 6 || "Mật khẩu phải có ít nhất 6 ký tự",
      ],
    };
  },

  mounted() {
    // Lấy user_id từ route
    this.userId = this.$route.params.id;

    // Lấy thông tin người dùng hiện tại
    this.fetchUserInfo();
  },

  methods: {
    async fetchUserInfo() {
      try {
        // Lấy thông tin user từ localStorage hoặc API
        const userLogin = JSON.parse(localStorage.getItem("userLogin"));

        if (!userLogin) {
          // Nếu không có thông tin user, chuyển về trang đăng nhập
          this.$router.push("/login");
          return;
        }

        // Kiểm tra xem user_id có khớp với user đang đăng nhập không
        if (this.userId != userLogin.MaDocGia) {
          this.snackbar.text = "Bạn không có quyền truy cập thông tin này";
          this.snackbar.color = "error";
          this.snackbar.show = true;
          this.$router.push("/");
          return;
        }

        // Gán thông tin user
        this.userInfo = {
          TenDangNhap: userLogin.TenDangNhap || "",
          HoTen: userLogin.HoTen || "",
          Email: userLogin.Email || "",
          SoDienThoai: userLogin.SoDienThoai || "",
        };
      } catch (error) {
        console.error("Lỗi khi lấy thông tin người dùng:", error);
        this.snackbar.text = "Không thể lấy thông tin người dùng";
        this.snackbar.color = "error";
        this.snackbar.show = true;
      }
    },

    async saveUserInfo() {
      // Kiểm tra form hợp lệ
      if (!this.$refs.form.validate()) {
        return;
      }

      this.loading = true;

      try {
        // Tạo request body
        const requestBody = {
          TenDangNhap: this.userInfo.TenDangNhap,
          HoTen: this.userInfo.HoTen,
          Email: this.userInfo.Email,
          SoDienThoai: this.userInfo.SoDienThoai,
        };

        // Thêm thông tin mật khẩu nếu người dùng muốn đổi
        if (this.passwordData.MatKhau && this.passwordData.NewPassword) {
          requestBody.MatKhau = this.passwordData.MatKhau;
          requestBody.NewPassword = this.passwordData.NewPassword;
          requestBody.ConfirmNewPassword = this.passwordData.ConfirmNewPassword;
        }

        // Gọi API cập nhật thông tin
        const response = await fetch(
          `http://26.193.242.15:8000/bandoc/update_user?user_id=${this.userId}`,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(requestBody),
          }
        );

        const data = await response.json();

        if (!response.ok) {
          throw new Error(data.message || "Không thể cập nhật thông tin");
        }

        // Cập nhật thông tin user trong localStorage
        const userLogin = JSON.parse(localStorage.getItem("userLogin"));
        userLogin.HoTen = this.userInfo.HoTen;
        userLogin.Email = this.userInfo.Email;
        userLogin.SoDienThoai = this.userInfo.SoDienThoai;
        localStorage.setItem("userLogin", JSON.stringify(userLogin));

        // Reset form mật khẩu
        this.passwordData = {
          MatKhau: "",
          NewPassword: "",
          ConfirmNewPassword: "",
        };

        // Hiển thị thông báo thành công
        this.snackbar.text = "Cập nhật thông tin thành công";
        this.snackbar.color = "success";
        this.snackbar.show = true;
      } catch (error) {
        console.error("Lỗi khi cập nhật thông tin:", error);
        this.snackbar.text =
          error.message || "Có lỗi xảy ra khi cập nhật thông tin";
        this.snackbar.color = "error";
        this.snackbar.show = true;
      } finally {
        this.loading = false;
      }
    },

    logout() {
      localStorage.removeItem("userLogin");
      this.$router.push("/login");
    },
  },
};
</script>

<style scoped>
@media (max-width: 600px) {
  .v-col {
    padding-bottom: 8px;
    padding-top: 8px;
  }
}
</style>
