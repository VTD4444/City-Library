<template>
  <div class="d-flex align-center justify-center">
    <v-container fluid>
      <v-row align="center" justify="center">
        <v-col cols="12" md="8" lg="6">
          <div class="text-center my-4">
            <p class="text-h4 font-weight-bold">ĐĂNG NHẬP</p>
          </div>

          <v-form v-model="valid" @submit.prevent="onSubmit">
            <v-row>
              <v-col cols="12" md="8" lg="6" class="mx-auto">
                <v-text-field
                  v-model="username"
                  :rules="usernameRules"
                  label="Tên đăng nhập *"
                  required
                  variant="outlined"
                  density="comfortable"
                  class="mb-3"
                >
                </v-text-field>

                <v-text-field
                  v-model="password"
                  :append-inner-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                  :type="showPassword ? 'text' : 'password'"
                  label="Mật khẩu *"
                  variant="outlined"
                  density="comfortable"
                  @click:append-inner="showPassword = !showPassword"
                  :rules="passwordRules"
                  required
                  class="mb-4"
                >
                </v-text-field>

                <v-alert
                  v-if="errorMessage"
                  type="error"
                  variant="tonal"
                  class="mb-4"
                  closable
                  @click:close="errorMessage = ''"
                >
                  {{ errorMessage }}
                </v-alert>

                <div class="d-flex justify-center gap-4 mt-4">
                  <v-btn
                    type="submit"
                    color="success"
                    size="large"
                    :loading="loading"
                    :disabled="!valid || loading"
                    min-width="150"
                    elevation="0"
                  >
                    {{ loading ? "Đang xử lý..." : "Đăng nhập" }}
                  </v-btn>
                </div>
              </v-col>
            </v-row>
          </v-form>
        </v-col>
      </v-row>
    </v-container>
  </div>

  <!-- Snackbar thông báo -->
  <v-snackbar v-model="snackbar" :color="snackbarColor" :timeout="3000">
    {{ snackbarText }}

    <template v-slot:actions>
      <v-btn icon @click="snackbar = false">
        <v-icon>mdi-close</v-icon>
      </v-btn>
    </template>
  </v-snackbar>
</template>

<script>
export default {
  layout: "default",
  data() {
    return {
      valid: false,
      username: "",
      password: "",
      showPassword: false,
      loading: false,
      errorMessage: "",
      usernameRules: [
        (v) => !!v || "Tên đăng nhập không được để trống",
        (v) => v.length >= 3 || "Tên đăng nhập phải có ít nhất 3 ký tự",
      ],
      passwordRules: [
        (v) => !!v || "Mật khẩu không được để trống",
        (v) => v.length >= 6 || "Mật khẩu phải có ít nhất 6 ký tự",
      ],
      snackbar: false,
      snackbarText: "",
      snackbarColor: "success",
    };
  },
  methods: {
    async onSubmit() {
      if (this.valid) {
        this.loading = true;
        this.errorMessage = "";

        try {
          // Tạo đối tượng dữ liệu gửi đến API
          const loginData = {
            TenDangNhap: this.username,
            MatKhau: this.password,
          };

          // Gọi API
          const response = await fetch(
            "https://26.193.242.15:8080/nhanvien/login",
            {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
                Accept: "application/json",
              },
              body: JSON.stringify(loginData),
            }
          );

          // Xử lý response
          const result = await response.json();

          if (response.ok) {
            // Hiển thị thông báo thành công
            this.snackbarText = "Đăng nhập thành công!";
            this.snackbarColor = "success";
            this.snackbar = true;

            // Lưu thông tin người dùng, token và ID người dùng vào localStorage
            localStorage.setItem("adminLogin", JSON.stringify(result));

            // Chuyển hướng đến trang chủ sau 1 giây
            setTimeout(() => {
              this.$router.push("/admin");
            }, 500);
          } else {
            // Hiển thị lỗi
            this.showError(
              "Đăng nhập thất bại: " +
                (result.message || "Tên đăng nhập hoặc mật khẩu không đúng")
            );
          }
        } catch (error) {
          console.error("Lỗi đăng nhập:", error);
          this.showError("Có lỗi xảy ra khi kết nối đến server");
        } finally {
          this.loading = false;
        }
      }
    },

    showError(message) {
      this.errorMessage = message;
    },
  },
};
</script>
