<template>
  <div>
    <!-- Desktop Header -->
    <div class="d-none d-md-block">
      <div class="mx-4 my-3">
        <div
          class="d-flex justify-space-between align-center py-2 mx-md-10 mx-2"
        >
          <div class="d-flex align-center">
            <img
              src="@/assets/images/logo.png"
              alt="Logo"
              height="40"
              width="40"
              class="mr-2"
            />
            <span class="text-h4 text-md-h4 text-sm-h5">City Library</span>
          </div>

          <!-- Phần đăng nhập hoặc chào người dùng -->
          <div class="mx-2">
            <!-- Hiển thị nếu chưa đăng nhập -->
            <template v-if="!isAuthenticated">
              <router-link
                to="/admin/login"
                class="text-decoration-none text-subtitle-1 text-green-darken-1"
                >Đăng nhập</router-link
              >
            </template>

            <!-- Hiển thị nếu đã đăng nhập -->
            <template v-else>
              <v-menu>
                <template v-slot:activator="{ props }">
                  <v-btn
                    v-bind="props"
                    rounded
                    variant="tonal"
                    color="green"
                    class="admin-menu-btn"
                  >
                    <v-icon left class="mr-1">mdi-account-circle</v-icon>
                    <span>Xin chào {{ HoTen }}</span>
                  </v-btn>
                </template>

                <v-list>
                  <v-list-item to="/admin/profile">
                    <v-list-item-title>Thông tin cá nhân</v-list-item-title>
                  </v-list-item>
                  <v-divider></v-divider>
                  <v-list-item @click="logout">
                    <v-list-item-title class="text-red"
                      >Đăng xuất</v-list-item-title
                    >
                  </v-list-item>
                </v-list>
              </v-menu>
            </template>
          </div>
        </div>

        <div
          class="border-t border-b d-flex justify-space-between align-center px-4 px-md-10"
        >
          <v-tabs v-model="activeTab" color="green-darken-1" align-tabs="start">
            <v-tab
              v-if="isAuthenticated"
              to="/admin"
              value="adminHome"
              class="text-subtitle-1 font-weight-medium"
              >TRANG CHỦ</v-tab
            >
            <v-tab
              v-if="isAuthenticated"
              to="/admin/manage-books"
              value="manageBooks"
              class="text-subtitle-1 font-weight-medium"
              >QUẢN LÝ TÀI LIỆU</v-tab
            >
            <v-tab
              v-if="isAuthenticated"
              to="/admin/genres"
              value="genres"
              class="text-subtitle-1 font-weight-medium"
              >THỂ LOẠI</v-tab
            >
            <v-tab
              v-if="isAuthenticated"
              to="/admin/authors"
              value="authors"
              class="text-subtitle-1 font-weight-medium"
              >TÁC GIẢ</v-tab
            >
            <v-tab
              v-if="isAuthenticated"
              to="/admin/readers"
              value="readers"
              class="text-subtitle-1 font-weight-medium"
              >BẠN ĐỌC</v-tab
            >
          </v-tabs>
        </div>
      </div>
    </div>

    <!-- Mobile Header -->
    <div class="d-md-none">
      <div class="mobile-header-spacer"></div>
      <v-app-bar
        color="white"
        elevation="0"
        app
        fixed
        class="safe-area-inset-top"
      >
        <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

        <div class="d-flex align-center">
          <img
            src="@/assets/images/logo.png"
            alt="Logo"
            height="32"
            width="32"
            class="mr-2"
          />
          <v-app-bar-title>City Library</v-app-bar-title>
        </div>

        <!-- Add login/logout to mobile header -->
        <v-spacer></v-spacer>
        <template v-if="isAuthenticated">
          <v-btn icon @click="logout">
            <v-icon>mdi-logout</v-icon>
          </v-btn>
        </template>
      </v-app-bar>

      <!-- Add a placeholder for the navigation drawer -->
      <v-navigation-drawer
        v-model="drawer"
        temporary
        :width="280"
        location="start"
        :elevation="4"
        class="mobile-navigation-drawer"
      >
        <div class="drawer-header pt-safe-area px-4 py-4 d-flex align-center">
          <img
            src="@/assets/images/logo.png"
            alt="Logo"
            height="32"
            width="32"
            class="mr-3"
          />
          <span class="text-h6">City Library</span>
          <v-spacer></v-spacer>
          <!-- Show admin name in drawer if authenticated -->
          <span v-if="isAuthenticated" class="text-body-2">{{ HoTen }}</span>
        </div>

        <v-divider></v-divider>

        <v-list density="comfortable" nav class="drawer-list px-2">
          <v-list-item to="/admin" exact color="green-darken-1">
            <template v-slot:prepend>
              <v-icon color="green-darken-1">mdi-home</v-icon>
            </template>
            <v-list-item-title>Trang chủ</v-list-item-title>
          </v-list-item>

          <v-list-item to="/admin/manage-books" color="green-darken-1">
            <template v-slot:prepend>
              <v-icon color="green-darken-1">mdi-book</v-icon>
            </template>
            <v-list-item-title>Quản lý tài liệu</v-list-item-title>
          </v-list-item>

          <v-list-item to="/admin/genres" color="green-darken-1">
            <template v-slot:prepend>
              <v-icon color="green-darken-1">mdi-bookmark</v-icon>
            </template>
            <v-list-item-title>Thể loại</v-list-item-title>
          </v-list-item>

          <v-list-item to="/admin/authors" color="green-darken-1">
            <template v-slot:prepend>
              <v-icon color="green-darken-1">mdi-account</v-icon>
            </template>
            <v-list-item-title>Tác giả</v-list-item-title>
          </v-list-item>

          <v-list-item to="/admin/readers" color="green-darken-1">
            <template v-slot:prepend>
              <v-icon color="green-darken-1">mdi-account-group</v-icon>
            </template>
            <v-list-item-title>Bạn đọc</v-list-item-title>
          </v-list-item>

          <!-- Add login/logout to mobile drawer -->
          <v-divider class="my-2"></v-divider>

          <!-- Show login if not authenticated -->
          <template v-if="!isAuthenticated">
            <v-list-item to="/admin/login" color="green-darken-1">
              <template v-slot:prepend>
                <v-icon color="green-darken-1">mdi-login</v-icon>
              </template>
              <v-list-item-title>Đăng nhập</v-list-item-title>
            </v-list-item>
          </template>

          <!-- Show logout if authenticated -->
          <template v-else>
            <v-list-item @click="logout" color="error">
              <template v-slot:prepend>
                <v-icon color="error">mdi-logout</v-icon>
              </template>
              <v-list-item-title class="text-red">Đăng xuất</v-list-item-title>
            </v-list-item>
          </template>
        </v-list>
      </v-navigation-drawer>
    </div>
  </div>
</template>

<script>
export default {
  name: "AdminHeader",
  data() {
    return {
      drawer: false,
      TenDangNhap: "",
      HoTen: "",
      MaDocGia: "",
      isAuthenticated: false,
      showDesktopNotifications: false,
      showMobileNotifications: false,
      notifications: [],
      unreadNotificationsCount: 0,
      loading: false,
    };
  },
  computed: {
    activeTab: {
      get() {
        const path = this.$route.path;
        if (path === "/admin") {
          return "adminHome";
        }
        if (path === "/admin/manage-books") {
          return "manageBooks";
        }
        if (path === "/admin/genres") {
          return "genres";
        }
        if (path === "/admin/authors") {
          return "authors";
        }
        if (path === "/admin/readers") {
          return "readers";
        }
        return null;
      },
    },
  },
  mounted() {
    // Check login status when component is mounted
    this.checkLoginStatus();
  },
  activated() {
    // Re-check login status when component is activated
    this.checkLoginStatus();
  },
  watch: {
    $route() {
      // Check login status on route change
      this.checkLoginStatus();
    },
  },
  methods: {
    // Xử lý đăng xuất
    logout() {
      // Xóa thông tin người dùng
      localStorage.removeItem("adminLogin");

      // Xóa thông tin admin trong component
      this.TenDangNhap = "";
      this.HoTen = "";
      this.MaDocGia = "";
      this.isAuthenticated = false;

      // Chuyển hướng đến trang đăng nhập
      this.$router.push("/admin/login");
    },

    // Thêm phương thức để kiểm tra đăng nhập và cập nhật UI
    checkLoginStatus() {
      // Check for admin login first
      let adminLogin = localStorage.getItem("adminLogin");
      if (adminLogin) {
        adminLogin = JSON.parse(adminLogin);
        this.isAuthenticated = true;
        this.TenDangNhap = adminLogin.TenDangNhap;
        this.MaDocGia = adminLogin.MaNV;
        this.HoTen = adminLogin.HoTen;
        return;
      }

      // Fallback to regular admin login
      let admin = localStorage.getItem("admin");
      if (admin) {
        adminLogin = JSON.parse(adminLogin);
        this.isAuthenticated = true;
        this.TenDangNhap = adminLogin.TenDangNhap;
        this.MaDocGia = adminLogin.MaDocGia;
        this.HoTen = adminLogin.HoTen;
      } else {
        this.isAuthenticated = false;
        this.TenDangNhap = "";
        this.HoTen = "";
        this.MaDocGia = "";
      }
    },
  },
};
</script>

<style scoped>
.border-t {
  border-top: 1px solid rgba(0, 0, 0, 0.12);
}

.border-b {
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

.text-red {
  color: #f44336 !important;
}

/* Add new styles for admin menu */
.admin-menu-btn {
  font-weight: normal;
}

/* Thêm style để xử lý hiển thị mobile */
.mobile-header-spacer {
  height: 56px; /* Chiều cao của app bar */
}

/* Hỗ trợ cho các thiết bị có notch như iPhone X trở lên */
.safe-area-inset-top {
  padding-top: env(safe-area-inset-top, 0);
  padding-top: constant(safe-area-inset-top, 0);
}

.pt-safe-area {
  padding-top: env(safe-area-inset-top, 20px);
  padding-top: constant(safe-area-inset-top, 20px);
}

/* Điều chỉnh nội dung chính để không bị che khuất bởi header */
:deep(.v-application__wrap) {
  padding-top: 56px; /* Chiều cao của app bar */
  min-height: 100vh;
  overflow: visible !important; /* Tránh cắt drawer */
}

@media (max-width: 600px) {
  .v-navigation-drawer {
    top: 0 !important;
    padding-top: env(safe-area-inset-top, 0);
  }
}

/* Navigation drawer fixes */
.navigation-drawer {
  z-index: 1000 !important;
  height: 100vh !important;
  position: fixed;
  top: 0;
  left: 0;
}

.drawer-content {
  padding-top: calc(56px + env(safe-area-inset-top, 0px));
  padding-bottom: env(safe-area-inset-bottom, 0px);
}

@media (max-width: 600px) {
  .navigation-drawer {
    height: 100% !important;
    position: fixed !important;
    top: 0 !important;
    left: 0 !important;
    transform: translateX(0) !important;
    padding-top: env(safe-area-inset-top, 0);
  }

  .v-navigation-drawer :deep(.v-list) {
    padding-top: 0;
  }

  /* Adjust for devices with notches */
  .drawer-content {
    padding-top: calc(56px + env(safe-area-inset-top, 0px));
    padding-bottom: env(safe-area-inset-bottom, 0px);
  }
}

/* Navigation drawer optimizations */
.mobile-navigation-drawer {
  position: fixed !important;
  top: 0 !important;
  z-index: 10000 !important;
}

.drawer-header {
  min-height: 56px;
  background-color: #f5f5f5;
}

.pt-safe-area {
  padding-top: env(safe-area-inset-top, 0);
  padding-top: constant(safe-area-inset-top, 0);
}

.drawer-list {
  margin-top: 8px;
}

/* Remove redundant styles */
.navigation-drawer,
.drawer-content {
  display: none;
}

@media (max-width: 600px) {
  .mobile-navigation-drawer {
    width: 280px !important;
    max-width: 85vw !important;
    height: 100vh !important; /* Force full viewport height */
    max-height: 100vh !important;
    overflow-y: auto;
  }

  /* Override Vuetify's default styles as needed */
  .mobile-navigation-drawer :deep(.v-navigation-drawer__content) {
    display: flex;
    flex-direction: column;
    overflow-x: hidden;
    overscroll-behavior: contain;
    height: 100vh !important; /* Ensure the content takes full height */
  }
}
</style>
