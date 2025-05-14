<template>
    <v-card 
        width="200" 
        height="350" 
        class="book-card mx-auto" 
        elevation="2"
        @click="viewBookDetail"
        :ripple="true"
    >
        <!-- Book Cover Image -->
        <v-img
            height="200"
            :src="book.coverImage || getImageUrl()"
            cover
            class="align-end"
        >
            <div class="category-chip">
                <v-chip
                    size="small"
                    color="white"
                    style="background-color: rgba(33, 33, 33, 0.85);"
                >
                    {{ book.category || book.TheLoai }}
                </v-chip>
            </div>
        </v-img>

        <!-- Book Information -->
        <v-card-title class="text-subtitle-1 py-2 text-truncate" :title="book.title || book.TieuDe">
            {{ book.title || book.TieuDe }}
        </v-card-title>
        
        <v-card-text class="py-0">
            <div class="text-body-2 mb-1 text-truncate">Tác giả: {{ book.author || book.TacGia }}</div>
            <div class="text-body-2 mb-1 text-truncate">Năm xuất bản: {{ book.publishYear || book.NamXuatBan }}</div>
            <div class="text-body-2 text-truncate">NXB: {{ book.publisher || book.NhaXuatBan }}</div>
        </v-card-text>

        <v-card-actions class="pt-0">
            <v-spacer></v-spacer>
            <v-btn
                variant="text"
                color="primary"
                size="small"
                class="text-lowercase"
                @click.stop="viewBookDetail"
            >
                Xem chi tiết
                <v-icon end icon="mdi-chevron-right"></v-icon>
            </v-btn>
        </v-card-actions>
    </v-card>
</template>

<script>
export default {
    name: 'BookCard',
    props: {
        book: {
            type: Object,
            required: true,
            default: () => ({
                id: '',
                MaSach: '',
                title: '',
                TieuDe: '',
                author: '',
                TacGia: '',
                publishYear: '',
                NamXuatBan: '',
                publisher: '',
                NXB: '',
                category: '',
                TheLoai: '',
                coverImage: '',
                AnhMinhHoaURL: ''
            })
        }
    },
    methods: {
        viewBookDetail() {
            // Lấy ID sách (hỗ trợ cả API cũ và mới)
            const bookId = this.book.id || this.book.MaSach;
            
            if (bookId) {
                // Chuyển hướng đến trang chi tiết sách
                this.$router.push(`/books/${bookId}`);
            }
        },
        getImageUrl() {
            // Xử lý đường dẫn ảnh từ API
            if (!this.book.AnhMinhHoaURL) {
                return 'https://www.svgrepo.com/show/508699/landscape-placeholder.svg';
            }
            
            if (this.book.AnhMinhHoaURL.startsWith('http')) {
                return this.book.AnhMinhHoaURL;
            }
            
            return `https://26.193.242.15:8080${this.book.AnhMinhHoaURL}`;
        }
    }
}
</script>

<style scoped>
.book-card {
    transition: transform 0.3s;
    cursor: pointer; /* Thêm con trỏ để hiển thị card có thể nhấp */
}

.book-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 6px 12px rgba(0,0,0,0.15) !important;
}

.category-chip {
    position: absolute;
    top: 8px;
    left: 8px;
}

/* Đảm bảo tiêu đề sách không bị tràn */
.v-card-title {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
</style>