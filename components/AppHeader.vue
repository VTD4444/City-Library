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

          <!-- Phần đăng nhập/đăng ký hoặc chào người dùng -->
          <div class="mx-2">
            <!-- Hiển thị nếu chưa đăng nhập -->
            <template v-if="!isAuthenticated">
              <router-link
                to="/login"
                class="text-decoration-none text-subtitle-1 text-green-darken-1"
                >Đăng nhập</router-link
              >
              <span
                class="mx-2"
                style="
                  display: inline-block;
                  width: 2px;
                  height: 20px;
                  background-color: #ccc;
                  vertical-align: middle;
                "
              ></span>
              <router-link
                to="/register"
                class="text-decoration-none text-subtitle-1 text-green-darken-1"
                >Đăng ký</router-link
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
                    class="user-menu-btn"
                  >
                    <v-icon left class="mr-1">mdi-account-circle</v-icon>
                    <span>Xin chào {{ HoTen }}</span>
                  </v-btn>
                </template>

                <v-list>
                  <v-list-item :to="'/user/' + MaDocGia">
                    <v-list-item-title>Tài khoản của tôi</v-list-item-title>
                  </v-list-item>
                  <v-list-item to="/user/borrow-tickets">
                    <v-list-item-title>Lịch sử mượn sách</v-list-item-title>
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
              to="/"
              value="home"
              class="text-subtitle-1 font-weight-medium"
              >TRANG CHỦ</v-tab
            >
            <v-tab
              to="/books"
              value="books"
              class="text-subtitle-1 font-weight-medium"
              >SÁCH</v-tab
            >
          </v-tabs>
          <div>
            <v-btn
              icon
              elevation="0"
              to="/shopping-cart"
              v-if="isAuthenticated"
            >
              <i class="fa-solid fa-cart-plus" style="color: #4b7c29"></i>
            </v-btn>

            <v-menu
              v-model="showDesktopNotifications"
              :close-on-content-click="false"
              location="bottom end"
              v-if="isAuthenticated"
            >
              <template v-slot:activator="{ props }">
                <v-btn icon elevation="0" v-bind="props" class="ml-2">
                  <v-badge
                    :content="
                      unreadNotificationsCount > 0
                        ? unreadNotificationsCount
                        : ''
                    "
                    :model-value="unreadNotificationsCount > 0"
                  >
                    <i class="fa-solid fa-bell" style="color: #4b7c29"></i>
                  </v-badge>
                </v-btn>
              </template>

              <v-card min-width="350" max-width="350" class="notification-panel">
                <v-card-title
                  class="d-flex justify-space-between align-center py-3 px-4"
                >
                  <span class="text-subtitle-1 font-weight-bold">Thông báo</span>
                  <v-btn
                    v-if="notifications.length > 0"
                    size="small"
                    variant="text"
                    color="primary"
                    @click="markAllAsRead"
                  >
                    Đánh dấu tất cả đã đọc
                  </v-btn>
                </v-card-title>

                <v-divider></v-divider>

                <div
                  class="notification-list"
                  :style="{ 'max-height': '400px', 'overflow-y': 'auto' }"
                >
                  <div
                    v-if="loading"
                    class="pa-4 text-center"
                  >
                    <v-progress-circular indeterminate color="primary"></v-progress-circular>
                    <div class="mt-2">Đang tải thông báo...</div>
                  </div>
                  
                  <div
                    v-else-if="notifications.length === 0"
                    class="pa-4 text-center grey--text"
                  >
                    <v-icon size="48" color="grey-lighten-1" class="mb-2"
                      >mdi-bell-off-outline</v-icon
                    >
                    <div>Bạn chưa có thông báo nào</div>
                  </div>

                  <template v-else>
                    <v-list lines="two" density="compact">
                      <v-list-item
                        v-for="notification in notifications"
                        :key="notification.MaThongBao"
                        :class="{
                          'unread-notification': notification.TrangThai === 'ChuaDoc',
                        }"
                        @click="viewNotification(notification)"
                      >
                        <v-list-item-title class="text-subtitle-2">
                          <v-icon
                            size="small"
                            :color="getNotificationIconColor(notification.LoaiThongBao)"
                            class="mr-1"
                          >
                            {{ getNotificationIcon(notification.LoaiThongBao) }}
                          </v-icon>
                          {{ getNotificationTitle(notification.LoaiThongBao) }}
                        </v-list-item-title>

                        <v-list-item-subtitle class="text-body-2 mt-1">
                          {{ notification.NoiDung }}
                        </v-list-item-subtitle>

                        <v-list-item-subtitle class="text-caption mt-1 text-grey">
                          {{ formatDate(notification.NgayTao) }}
                        </v-list-item-subtitle>
                      </v-list-item>
                    </v-list>
                  </template>
                </div>
              </v-card>
            </v-menu>
          </div>
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

        <v-spacer></v-spacer>

        <v-btn icon elevation="0" to="/shopping-cart" v-if="isAuthenticated">
          <i class="fa-solid fa-cart-plus" style="color: #4b7c29"></i>
        </v-btn>

        <v-menu
          v-model="showMobileNotifications"
          :close-on-content-click="false"
          location="bottom end"
          v-if="isAuthenticated"
        >
          <template v-slot:activator="{ props }">
            <v-btn icon elevation="0" v-bind="props" class="ml-2">
              <v-badge
                :content="
                  unreadNotificationsCount > 0 ? unreadNotificationsCount : ''
                "
                :model-value="unreadNotificationsCount > 0"
              >
                <i class="fa-solid fa-bell" style="color: #4b7c29"></i>
              </v-badge>
            </v-btn>
          </template>

          <v-card min-width="350" max-width="350" class="notification-panel">
            <v-card-title
              class="d-flex justify-space-between align-center py-3 px-4"
            >
              <span class="text-subtitle-1 font-weight-bold">Thông báo</span>
              <v-btn
                v-if="notifications.length > 0"
                size="small"
                variant="text"
                color="primary"
                @click="markAllAsRead"
              >
                Đánh dấu tất cả đã đọc
              </v-btn>
            </v-card-title>

            <v-divider></v-divider>

            <div
              class="notification-list"
              :style="{ 'max-height': '400px', 'overflow-y': 'auto' }"
            >
              <div
                v-if="loading"
                class="pa-4 text-center"
              >
                <v-progress-circular indeterminate color="primary"></v-progress-circular>
                <div class="mt-2">Đang tải thông báo...</div>
              </div>
              
              <div
                v-else-if="notifications.length === 0"
                class="pa-4 text-center grey--text"
              >
                <v-icon size="48" color="grey-lighten-1" class="mb-2"
                  >mdi-bell-off-outline</v-icon
                >
                <div>Bạn chưa có thông báo nào</div>
              </div>

              <template v-else>
                <v-list lines="two" density="compact">
                  <v-list-item
                    v-for="notification in notifications"
                    :key="notification.MaThongBao"
                    :class="{
                      'unread-notification': notification.TrangThai === 'ChuaDoc',
                    }"
                    @click="viewNotification(notification)"
                  >
                    <v-list-item-title class="text-subtitle-2">
                      <v-icon
                        size="small"
                        :color="getNotificationIconColor(notification.LoaiThongBao)"
                        class="mr-1"
                      >
                        {{ getNotificationIcon(notification.LoaiThongBao) }}
                      </v-icon>
                      {{ getNotificationTitle(notification.LoaiThongBao) }}
                    </v-list-item-title>

                    <v-list-item-subtitle class="text-body-2 mt-1">
                      {{ notification.NoiDung }}
                    </v-list-item-subtitle>

                    <v-list-item-subtitle class="text-caption mt-1 text-grey">
                      {{ formatDate(notification.NgayTao) }}
                    </v-list-item-subtitle>
                  </v-list-item>
                </v-list>
              </template>
            </div>

            <v-divider></v-divider>

            <v-card-actions class="justify-center py-2">
              <v-btn
                variant="text"
                color="primary"
                to="/notifications"
                @click="showMobileNotifications = false"
              >
                Xem tất cả thông báo
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
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
        </div>
        
        <v-divider></v-divider>
        
        <v-list density="comfortable" nav class="drawer-list px-2">
          <v-list-item to="/" exact color="green-darken-1">
            <template v-slot:prepend>
              <v-icon color="green-darken-1">mdi-home</v-icon>
            </template>
            <v-list-item-title>Trang chủ</v-list-item-title>
          </v-list-item>

          <v-list-item to="/books" color="green-darken-1">
            <template v-slot:prepend>
              <v-icon color="green-darken-1">mdi-book-open-variant</v-icon>
            </template>
            <v-list-item-title>Sách</v-list-item-title>
          </v-list-item>

          <v-divider class="my-2"></v-divider>

          <!-- Menu cho user chưa đăng nhập -->
          <template v-if="!isAuthenticated">
            <v-list-item to="/login" color="green-darken-1">
              <template v-slot:prepend>
                <v-icon color="green-darken-1">mdi-login</v-icon>
              </template>
              <v-list-item-title>Đăng nhập</v-list-item-title>
            </v-list-item>

            <v-list-item to="/register" color="green-darken-1">
              <template v-slot:prepend>
                <v-icon color="green-darken-1">mdi-account-plus</v-icon>
              </template>
              <v-list-item-title>Đăng ký</v-list-item-title>
            </v-list-item>
          </template>

          <!-- Menu cho user đã đăng nhập -->
          <template v-else>
            <v-list-item :to="'/user/' + MaDocGia" color="green-darken-1">
              <template v-slot:prepend>
                <v-icon color="green-darken-1">mdi-account-circle</v-icon>
              </template>
              <v-list-item-title>Tài khoản của tôi</v-list-item-title>
            </v-list-item>

            <v-list-item to="/user/borrow-tickets" color="green-darken-1">
              <template v-slot:prepend>
                <v-icon color="green-darken-1">mdi-book-open-page-variant</v-icon>
              </template>
              <v-list-item-title>Lịch sử mượn sách</v-list-item-title>
            </v-list-item>

            <v-divider class="my-2"></v-divider>

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
  name: "AppHeader",
  data() {
    return {
      // Add these new properties
      notificationInterval: null,
      notificationPollingTime: 30000, // 30 seconds refresh interval
      refreshing: false,
      
      // Your existing data properties
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
        if (path === "/") {
          return "home";
        }
        if (path === "/books" || path.startsWith("/books/")) {
          return "books";
        }
        return null;
      },
      set(value) {
        // Setter required for v-model
      },
    },
  },

  methods: {
    // Xử lý đăng xuất
    logout() {
      // Stop notification polling
      this.stopNotificationPolling();

      // Xóa thông tin người dùng
      localStorage.removeItem("userLogin");

      // Xóa thông tin user trong component
      this.TenDangNhap = "";
      this.HoTen = "";
      this.MaDocGia = "";
      this.isAuthenticated = false;

      // Hiển thị thông báo (nếu cần)
      this.$emit("logout-success");

      // Chuyển hướng đến trang chủ
      this.$router.push("/");
    },

    // Thêm phương thức để kiểm tra đăng nhập và cập nhật UI
    checkLoginStatus() {
      let userLogin = localStorage.getItem("userLogin");
      if (userLogin) {
        userLogin = JSON.parse(userLogin);
        this.isAuthenticated = true;
        this.TenDangNhap = userLogin.TenDangNhap;
        this.MaDocGia = userLogin.MaDocGia;
        this.HoTen = userLogin.HoTen;
      } else {
        this.isAuthenticated = false;
        this.TenDangNhap = "";
        this.HoTen = "";
        this.MaDocGia = "";
      }
    },

    async refreshNotifications() {
      if (this.refreshing || !this.isAuthenticated) return;
      
      this.refreshing = true;
      try {
        await this.loadNotifications();
      } finally {
        this.refreshing = false;
      }
    },

    async loadNotifications(silent = false) {
      if (!this.isAuthenticated || !this.MaDocGia) return;
      
      if (!silent) this.loading = true;
      try {
        const response = await fetch(`http://26.193.242.15:8000/thongbao/${this.MaDocGia}`);
        if (!response.ok) {
          throw new Error(`API error! Status: ${response.status}`);
        }
        
        const newNotifications = await response.json();
        
        // Check if there are any new notifications to highlight
        if (this.notifications.length > 0) {
          const existingIds = new Set(this.notifications.map(n => n.MaThongBao));
          const hasNewNotifications = newNotifications.some(n => !existingIds.has(n.MaThongBao));
          
          if (hasNewNotifications) {
            this.showNewNotificationAlert();
          }
        }
        
        this.notifications = newNotifications;
        
        // Đếm số lượng thông báo chưa đọc
        this.unreadNotificationsCount = this.notifications.filter(
          (item) => item.TrangThai === "ChuaDoc"
        ).length;
        
      } catch (error) {
        console.error("Error loading notifications:", error);
        if (!silent) {
          this.notifications = [];
          this.unreadNotificationsCount = 0;
        }
      } finally {
        if (!silent) this.loading = false;
      }
    },

    async viewNotification(notification) {
      if (notification.TrangThai === "ChuaDoc") {
        try {
          // Gọi API để đánh dấu là đã đọc
          const response = await fetch(`http://26.193.242.15:8000/thongbao/xacnhan/${notification.MaThongBao}`, {
            method: 'GET',
            headers: {
              'Accept': 'application/json'
            }
          });
          
          if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.message || `HTTP error! Status: ${response.status}`);
          }
          
          // Cập nhật trạng thái trong mảng hiện tại
          notification.TrangThai = "DaDoc";
          
          // Cập nhật số lượng thông báo chưa đọc
          this.unreadNotificationsCount = this.notifications.filter(
            (item) => item.TrangThai === "ChuaDoc"
          ).length;
        } catch (error) {
          console.error("Error marking notification as read:", error);
          // Có thể hiển thị thông báo lỗi nếu cần thiết
        }
      }

      // Xử lý chuyển hướng dựa vào loại thông báo
      if (
        notification.LoaiThongBao === "PHIEU_TAO_THANH_CONG" &&
        notification.MaPhieuMuon
      ) {
        this.$router.push(`/user/borrow-tickets/${notification.MaPhieuMuon}`);
        this.showDesktopNotifications = false;
        this.showMobileNotifications = false;
      }
    },

    async markAllAsRead() {
      if (this.notifications.length === 0 || !this.MaDocGia) return;
      
      const unreadNotifications = this.notifications.filter(n => n.TrangThai === "ChuaDoc");
      if (unreadNotifications.length === 0) return;
      
      try {
        // Gọi API để đánh dấu tất cả thông báo là đã đọc
        const promises = unreadNotifications.map(notification => 
          fetch(`http://26.193.242.15:8000/thongbao/xacnhan/${notification.MaThongBao}`, {
            method: 'GET',
            headers: {
              'Accept': 'application/json'
            }
          })
        );
        
        await Promise.all(promises);
        
        // Cập nhật trạng thái trong mảng hiện tại
        this.notifications.forEach(notification => {
          notification.TrangThai = "DaDoc";
        });
        
        // Cập nhật số lượng thông báo chưa đọc
        this.unreadNotificationsCount = 0;
        
        // Có thể thêm thông báo thành công nếu cần thiết
      } catch (error) {
        console.error("Error marking all notifications as read:", error);
        // Có thể hiển thị thông báo lỗi nếu cần thiết
      }
    },

    getNotificationIcon(type) {
      switch (type) {
        case "PHIEU_TAO_THANH_CONG":
          return "mdi-bookmark-check-outline";
        case "PHIEU_XAC_NHAN":
          return "mdi-check-circle-outline";
        case "PHIEU_TU_CHOI":
          return "mdi-close-circle-outline";
        default:
          return "mdi-bell-outline";
      }
    },

    getNotificationIconColor(type) {
      switch (type) {
        case "PHIEU_TAO_THANH_CONG":
          return "success";
        case "PHIEU_XAC_NHAN":
          return "info";
        case "PHIEU_TU_CHOI":
          return "error";
        default:
          return "grey";
      }
    },

    getNotificationTitle(type) {
      switch (type) {
        case "PHIEU_TAO_THANH_CONG":
          return "Tạo phiếu mượn";
        case "PHIEU_XAC_NHAN":
          return "Phiếu được xác nhận";
        case "PHIEU_TU_CHOI":
          return "Phiếu bị từ chối";
        default:
          return "Thông báo";
      }
    },

    formatDate(dateString) {
      if (!dateString) return "";

      const date = new Date(dateString);
      const today = new Date();

      // Nếu là hôm nay
      if (date.toDateString() === today.toDateString()) {
        return `Hôm nay, ${date.getHours().toString().padStart(2, "0")}:${date
          .getMinutes()
          .toString()
          .padStart(2, "0")}`;
      }

      // Nếu là ngày hôm qua
      const yesterday = new Date();
      yesterday.setDate(today.getDate() - 1);
      if (date.toDateString() === yesterday.toDateString()) {
        return `Hôm qua, ${date.getHours().toString().padStart(2, "0")}:${date
          .getMinutes()
          .toString()
          .padStart(2, "0")}`;
      }

      // Ngày khác
      return `${date.getDate().toString().padStart(2, "0")}/${(
        date.getMonth() + 1
      )
        .toString()
        .padStart(2, "0")}/${date.getFullYear()}, ${date
        .getHours()
        .toString()
        .padStart(2, "0")}:${date.getMinutes().toString().padStart(2, "0")}`;
    },

    startNotificationPolling() {
      // Clear any existing interval first
      this.stopNotificationPolling();
      
      // Set up a new interval
      this.notificationInterval = setInterval(() => {
        this.loadNotifications(true); // true = silent loading
      }, this.notificationPollingTime);
    },
    
    stopNotificationPolling() {
      if (this.notificationInterval) {
        clearInterval(this.notificationInterval);
        this.notificationInterval = null;
      }
    },
    
    showNewNotificationAlert() {
      // You could play a sound, show a toast, or other indicator
      console.log("New notifications available!");
      
      // If you want to use a sound:
      // const notificationSound = new Audio('/notification-sound.mp3');
      // notificationSound.play();
    },
  },

  // Gọi API lấy thông tin người dùng khi component được tạo
  mounted() {
    this.checkLoginStatus();
    if (this.isAuthenticated) {
      this.loadNotifications();
      this.startNotificationPolling(); // Start polling
    }
    this.$root.$on("new-notification", this.loadNotifications);
  },

  // Cập nhật sau mỗi lần hiển thị trang (sửa phần thiếu này)
  activated() {
    this.checkLoginStatus();
  },

  beforeUnmount() {
    // Stop polling when component unmounts
    this.stopNotificationPolling();
    // Xóa event listener khi component unmount
    this.$root.$off("new-notification", this.loadNotifications);
  },

  // Theo dõi thay đổi của route để cập nhật lại thông tin khi cần
  watch: {
    $route() {
      // Kiểm tra lại trạng thái đăng nhập mỗi khi chuyển trang
      this.checkLoginStatus();
    },
    isAuthenticated(newVal) {
      if (newVal === true) {
        this.loadNotifications();
        this.startNotificationPolling(); // Start polling when user logs in
      } else {
        this.stopNotificationPolling(); // Stop polling when user logs out
      }
    },
    MaDocGia(newVal) {
      if (newVal && this.isAuthenticated) {
        this.loadNotifications();
      }
    }
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

.user-menu-btn {
  font-weight: normal;
}

.notification-panel {
  box-shadow: 0 4px 25px 0 rgba(0, 0, 0, 0.1);
}

.unread-notification {
  background-color: rgba(66, 165, 245, 0.05);
  position: relative;
}

.unread-notification::before {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 70%;
  background-color: #2196f3;
  border-radius: 0 4px 4px 0;
}

.notification-list :deep(.v-list-item) {
  min-height: 70px;
  padding: 12px 16px;
  cursor: pointer;
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
