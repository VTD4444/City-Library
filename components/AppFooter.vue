<template>
  <v-footer class="bg-green-darken-3 text-white">
    <v-container>
      <v-row class="py-6">
        <!-- Logo và tên -->
        <v-col cols="12" md="4" class="d-flex align-center">
          <v-img
            src="@/assets/images/logo.png"
            max-width="50"
            max-height="50"
            class="mr-3"
            alt="City Library Logo"
          ></v-img>
          <h2 class="text-h5">City Library</h2>
        </v-col>

        <!-- Thông tin -->
        <v-col cols="12" md="4">
          <h3 class="text-h6 mb-4">Thông tin</h3>
          <div class="mb-2">
            <a href="/gioi-thieu" class="text-decoration-none text-white">Giới thiệu</a>
          </div>
          <div class="mb-2">
            <a href="/tin-tuc" class="text-decoration-none text-white">Tin tức sự kiện</a>
          </div>
          <div>
            <a href="/sach-moi" class="text-decoration-none text-white">Sách mới</a>
          </div>
        </v-col>

        <!-- Liên hệ -->
        <v-col cols="12" md="4">
          <h3 class="text-h6 mb-4">Liên hệ</h3>
          <div class="mb-2">Địa chỉ: Km 10, Trần Phú, Hà Đông, Hà Nội</div>
          <div class="mb-2">Hotline: 0244782848</div>
          <div class="mb-2">Mạng xã hội:</div>
          <div class="d-flex">
            <v-btn
              icon
              variant="text"
              href="https://facebook.com/dinhvutrungduc"
              target="_blank"
              rel="noopener noreferrer"
            >
              <v-icon>mdi-facebook</v-icon>
            </v-btn>
            <v-btn
              icon
              variant="text"
              href="https://instagram.com/vtd_4444"
              target="_blank"
              rel="noopener noreferrer"
            >
              <v-icon>mdi-instagram</v-icon>
            </v-btn>
          </div>
        </v-col>
      </v-row>

      <v-divider class="border-opacity-25"></v-divider>

      <div class="py-3 text-center text-md-left">
        <v-icon size="small" class="mr-1">mdi-copyright</v-icon>
        2025 City Library Ltd
      </div>
    </v-container>
  </v-footer>
</template>

<script>
export default {
  name: 'AppFooter'
}
</script>

<style scoped>
a:hover {
  text-decoration: underline !important;
  opacity: 0.8;
}
</style>