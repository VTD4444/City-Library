<template>
  <div>
    <div class="d-flex align-center justify-center">
      <v-container fluid>
        <v-row align="center" justify="center">
          <v-col cols="12" md="10" lg="8">
              <div class="text-center my-4">
                <p class="text-h4 font-weight-bold">ĐĂNG KÝ THẺ THƯ VIỆN</p>
              </div>
              
              <v-form ref="form" v-model="valid" @submit.prevent="onSubmit">
                <v-row>
                  <!-- Cột trái: Thông tin cá nhân -->
                  <v-col cols="12" md="6">
                    <p class="text-h6 font-weight-bold mb-4">Thông tin cá nhân:</p>

                    <v-text-field
                      v-model="TenDangNhap"
                      :rules="tenDangNhapRules"
                      label="Tài khoản *"
                      required
                      variant="outlined"
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>

                    <v-text-field
                      v-model="HoTen"
                      :rules="hoTenRules"
                      label="Họ tên *"
                      required
                      variant="outlined"
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>

                    <v-text-field
                      v-model="NgaySinh"
                      label="Ngày sinh *"
                      type="date"
                      required
                      variant="outlined"
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>

                    <v-text-field
                      v-model="MatKhau"
                      :append-inner-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                      :type="showPassword ? 'text' : 'password'"
                      label="Mật khẩu *"
                      variant="outlined"
                      @click:append-inner="showPassword = !showPassword"
                      :rules="matKhauRules"
                      required
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>

                    <v-text-field
                      v-model="ConfirmPassword"
                      :append-inner-icon="showConfirmPassword ? 'mdi-eye' : 'mdi-eye-off'"
                      :type="showConfirmPassword ? 'text' : 'password'"
                      label="Xác nhận mật khẩu *"
                      variant="outlined"
                      @click:append-inner="showConfirmPassword = !showConfirmPassword"
                      :rules="confirmPasswordRules"
                      required
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>
                  </v-col>
                  
                  <!-- Cột phải: Thông tin liên hệ -->
                  <v-col cols="12" md="6">
                    <p class="text-h6 font-weight-bold mb-4">Thông tin liên hệ:</p>
                    
                    <v-text-field
                      v-model="SoDienThoai"
                      :rules="soDienThoaiRules"
                      label="Số điện thoại *"
                      required
                      variant="outlined"
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>

                    <v-text-field
                      v-model="Email"
                      :rules="emailRules"
                      label="Email *"
                      required
                      variant="outlined"
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>

                    <v-text-field
                      v-model="DiaChi"
                      :rules="diaChiRules"
                      label="Địa chỉ *"
                      required
                      variant="outlined"
                      class="mb-2"
                      density="comfortable"
                    ></v-text-field>
                  </v-col>
                </v-row>

                <!-- Hiển thị lỗi nếu có -->
                <v-alert
                  v-if="errorMessage"
                  type="error"
                  variant="tonal"
                  class="mb-4"
                  closable
                >
                  {{ errorMessage }}
                </v-alert>

                <div class="d-flex justify-center gap-4 mt-4">
                    <v-btn
                    color="yellow-darken-3"
                    elevation="0"
                    size="large"
                    min-width="150"
                    class="mr-4 hover:bg-yellow-darken-5"
                    @click="resetForm"
                    >
                    Đặt lại
                  </v-btn>
                  
                  <v-btn
                    type="submit"
                    color="success"
                    elevation="0"
                    size="large"
                    :loading="loading"
                    :disabled="!valid || loading"
                    min-width="150"
                  >
                    {{ loading ? 'Đang xử lý...' : 'Đăng ký' }}
                  </v-btn>
                </div>

                <div class="text-center mt-4">
                  <span class="text-body-1">Đã có tài khoản? </span>
                  <router-link
                    to="/login"
                    class="text-green-darken-1 font-weight-bold text-decoration-none"
                  >
                    Đăng nhập ngay
                  </router-link>
                </div>
              </v-form>
          </v-col>
        </v-row>

        <!-- Thêm dialog thông báo đăng ký thành công -->
        <v-dialog
          v-model="successDialog"
          max-width="500px"
          persistent
        >
          <v-card class="pa-4">
            <v-card-title class="text-h5 text-center font-weight-bold">
              THÔNG BÁO
            </v-card-title>
            
            <v-card-text class="text-body-1 pt-4">
              <p class="mb-4">Đăng ký thẻ thư viện thành công.</p>
              
              <p class="mb-4 text-red">
                Trong thời gian 7 ngày, bạn đọc vui lòng đến City Library để được cấp thẻ. 
                Sau thời gian này, đăng ký sẽ không còn hiệu lực.
              </p>
              
              <p class="font-italic">City Library xin chân thành cảm ơn!</p>
            </v-card-text>
            
            <v-card-actions class="justify-center pt-2">
              <v-btn
                color="success"
                min-width="100"
                @click="closeSuccessDialog"
              >
                Đã hiểu
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
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
  </div>
</template>

<script>
export default {
  data() {
    return {
      valid: false,
      // Đổi tên field theo API
      HoTen: '',
      NgaySinh: '',
      DiaChi: '',
      SoDienThoai: '',
      Email: '',
      TenDangNhap: '',
      MatKhau: '',
      ConfirmPassword: '',
      showPassword: false,
      showConfirmPassword: false,
      loading: false,
      errorMessage: '',
      
      // Các rule validation
      tenDangNhapRules: [
        (v) => !!v || 'Tài khoản không được để trống',
        (v) => v.length >= 3 || 'Tài khoản phải có ít nhất 3 ký tự',
      ],
      hoTenRules: [
        (v) => !!v || 'Họ tên không được để trống',
        (v) => v.length >= 2 || 'Họ tên phải có ít nhất 2 ký tự',
      ],
      emailRules: [
        (v) => !!v || 'Email không được để trống',
        (v) => /.+@.+\..+/.test(v) || 'Email không hợp lệ',
      ],
      matKhauRules: [
        (v) => !!v || 'Mật khẩu không được để trống',
        (v) => v.length >= 6 || 'Mật khẩu phải có ít nhất 6 ký tự',
      ],
      confirmPasswordRules: [
        (v) => !!v || 'Xác nhận mật khẩu không được để trống',
        (v) => v === this.MatKhau || 'Mật khẩu xác nhận không khớp',
      ],
      soDienThoaiRules: [
        (v) => !!v || 'Số điện thoại không được để trống',
        (v) => /^\d{10}$/.test(v) || 'Số điện thoại phải có 10 chữ số',
      ],
      diaChiRules: [
        (v) => !!v || 'Địa chỉ không được để trống',
      ],
      
      snackbar: false,
      snackbarText: '',
      snackbarColor: 'success',
      
      // Thêm biến điều khiển dialog
      successDialog: false,
    };
  },
  methods: {
    async onSubmit() {
      if (this.valid) {
        this.loading = true;
        this.errorMessage = '';

        try {
          // Tạo đối tượng dữ liệu theo cấu trúc API
          const userData = {
            HoTen: this.HoTen,
            NgaySinh: this.NgaySinh,
            DiaChi: this.DiaChi,
            SoDienThoai: this.SoDienThoai,
            Email: this.Email,
            TenDangNhap: this.TenDangNhap,
            MatKhau: this.MatKhau,
            ConfirmPassword: this.ConfirmPassword
          };

          // Gọi API đăng ký
          const response = await fetch(
            'https://26.193.242.15:8080/bandoc/create',
            {
              method: 'POST',
              headers: {
                'Content-Type': 'application/json',
                'Accept': 'application/json',
              },
              body: JSON.stringify(userData),
            }
          );

          // Xử lý response
          const result = await response.json();

          if (response.ok) {
            // Hiển thị dialog thông báo thành công thay vì snackbar
            this.successDialog = true;
          } else {
            // Xử lý lỗi từ server
            this.snackbarText = result.detail || 'Đăng ký thất bại!';
            this.snackbarColor = 'error';
            this.snackbar = true;
          }
        } catch (error) {
          console.error('Lỗi đăng ký:', error);
          this.errorMessage = 'Có lỗi xảy ra khi kết nối đến server!';
        } finally {
          this.loading = false;
        }
      }
    },
    
    resetForm() {
      // Đặt lại tất cả các trường form
      this.HoTen = '';
      this.NgaySinh = '';
      this.DiaChi = '';
      this.SoDienThoai = '';
      this.Email = '';
      this.TenDangNhap = '';
      this.MatKhau = '';
      this.ConfirmPassword = '';
      this.errorMessage = '';
      
      // Reset validation nếu form đã được tạo
      if (this.$refs.form) {
        this.$refs.form.resetValidation();
      }
    },
    
    // Thêm phương thức đóng dialog và chuyển hướng
    closeSuccessDialog() {
      this.successDialog = false;
      this.$router.push('/login');
    },
  },
};
</script>

<style scoped>
/* Thêm style cho dialog nếu cần */
.v-card-title {
  color: #2e7d32;
  border-bottom: 1px solid #e0e0e0;
  padding-bottom: 12px;
}

.font-italic {
  font-style: italic;
}
</style>