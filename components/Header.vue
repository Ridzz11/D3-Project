<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light shadow-sm fixed-top">
      <div class="container-fluid">
        <button class="btn btn-outline-dark d-lg-none" @click="toggleSidebar">
          <i class="bi bi-list"></i>
        </button>
        <a class="navbar-brand mx-auto" href="#">
          <img src="../assets/img/logo.jpg" alt="Logo" height="30" class="me-2" />
          <span>PT. DTHREE Sukses Mulia</span>
        </a>
      </div>
    </nav>

    <div class="d-flex">
      <div :class="['sidebar', { 'sidebar-show': showSidebar || isLargeScreen }]">
        <div class="sticky-top bg-sidebar">
          <input type="text" class="form-control mb-3" placeholder="Search..." />
          <ul class="nav flex-column">
            <li class="nav-item">
              <p class="mb-1 text-dark fw-bold">Main Navigation</p>
              <hr />
            </li>
          </ul>
        </div>

        <div class="menu-scroll-area py-2 my-2">
          <ul class="nav flex-column">
            <li class="nav-item">
              <NuxtLink to="/" class="nav-link text-dark d-flex align-items-center">
                <i class="bi bi-speedometer2 me-2"></i> Dashboard
              </NuxtLink>
            </li>
            <li class="nav-item">
              <NuxtLink to="/databarang/" class="nav-link text-dark d-flex align-items-center">
                <i class="bi bi-box me-2"></i> Data Product
              </NuxtLink>
            </li>
            <li class="nav-item">
              <NuxtLink to="/data-kategori/" class="nav-link text-dark d-flex align-items-center">
                <i class="bi bi-box me-2"></i> Data Kategori
              </NuxtLink>
            </li>
            <li class="nav-item">
              <a href="#" class="nav-link text-dark d-flex align-items-center" @click.prevent="toggleKelolaBarang">
                <i class="bi bi-tools me-2"></i> Kelola Barang
                <i class="bi ms-auto" :class="showKelolaBarang ? 'bi-chevron-up' : 'bi-chevron-down'"></i>
              </a>
              <ul v-show="showKelolaBarang" class="nav flex-column ms-3">
                <li class="nav-item">
                  <a href="#" class="nav-link text-dark" @click.prevent="toggleGudang1">
                    <i class="bi bi-box-arrow-in-down me-2"></i> Gudang Lantai 1
                    <i class="bi ms-auto" :class="showGudang1 ? 'bi-chevron-up' : 'bi-chevron-down'"></i>
                  </a>
                  <ul v-show="showGudang1" class="nav flex-column ms-3">
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiOne/barang-masuk" class="nav-link text-dark">Barang Masuk</NuxtLink>
                    </li>
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiOne/barang-keluar" class="nav-link text-dark">Barang Keluar</NuxtLink>
                    </li>
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiOne/monitoring/" class="nav-link text-dark">Monitoring Gudang 1</NuxtLink>
                    </li>
                  </ul>
                </li>
                <li class="nav-item">
                  <a href="#" class="nav-link text-dark" @click.prevent="toggleGudang2">
                    <i class="bi bi-box-arrow-up me-2"></i> Gudang Lantai 2
                    <i class="bi ms-auto" :class="showGudang2 ? 'bi-chevron-up' : 'bi-chevron-down'"></i>
                  </a>
                  <ul v-show="showGudang2" class="nav flex-column ms-3">
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiTwo/barang-masuk" class="nav-link text-dark">Barang Masuk</NuxtLink>
                    </li>
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiTwo/barang-keluar" class="nav-link text-dark">Barang Keluar</NuxtLink>
                    </li>
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiTwo/monitoring/" class="nav-link text-dark">Monitoring Gudang 2</NuxtLink>
                    </li>
                  </ul>
                </li>
                <li class="nav-item">
                  <a href="#" class="nav-link text-dark" @click.prevent="toggleGudang3">
                    <i class="bi bi-box-arrow-up me-2"></i> Gudang Lantai 3
                    <i class="bi ms-auto" :class="showGudang3 ? 'bi-chevron-up' : 'bi-chevron-down'"></i>
                  </a>
                  <ul v-show="showGudang3" class="nav flex-column ms-3">
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiThree/barang-masuk" class="nav-link text-dark">Barang Masuk</NuxtLink>
                    </li>
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiThree/barang-keluar" class="nav-link text-dark">Barang Keluar</NuxtLink>
                    </li>
                    <li class="nav-item">
                      <NuxtLink to="/kelola-barang/G_LantaiThree/monitoring/" class="nav-link text-dark">Monitoring Gudang 3</NuxtLink>
                    </li>
                  </ul>
                </li>
              </ul>
            </li>
            <li class="nav-item">
              <NuxtLink to="/kelola-toko" class="nav-link text-dark d-flex align-items-center">
                <i class="bi bi-shop me-2"></i> Kelola Toko
              </NuxtLink>
            </li>
            <li class="nav-item">
              <a href="#" class="nav-link text-dark d-flex align-items-center" @click.prevent="handleLogout">
                <i class="bi bi-box-arrow-right me-2"></i> Logout
              </a>
            </li>
          </ul>
        </div>
      </div>

      <div class="p-5 mt-3" :class="['main-content', { 'content-shift': showSidebar || isLargeScreen }]">
        <slot />
      </div>
    </div>
  </div>
</template>

<script setup>
const supabase = useSupabaseClient();
const router = useRouter(); 

const showSidebar = ref(false);
const showKelolaBarang = ref(false);
const showGudang1 = ref(false);
const showGudang2 = ref(false);
const showGudang3 = ref(false);
const isLargeScreen = ref(false);

const toggleSidebar = () => {
  showSidebar.value = !showSidebar.value;
};
const toggleKelolaBarang = () => {
  showKelolaBarang.value = !showKelolaBarang.value;
};
const toggleGudang1 = () => {
  showGudang1.value = !showGudang1.value;
};
const toggleGudang2 = () => {
  showGudang2.value = !showGudang2.value;
};
const toggleGudang3 = () => {
  showGudang3.value = !showGudang3.value;
};

const handleResize = () => {
  if (typeof window !== "undefined") {
    isLargeScreen.value = window.innerWidth >= 992;
    if (isLargeScreen.value) showSidebar.value = false;
  }
};

onMounted(() => {
  handleResize();
  window.addEventListener("resize", handleResize);
});

onUnmounted(() => {
  window.removeEventListener("resize", handleResize);
});

const handleLogout = async () => {
  const { error } = await supabase.auth.signOut();
  if (!error) {
    router.push('/login'); 
  } else {
    console.error('Logout gagal:', error.message);
  }
};
</script>

<style scoped>
.sidebar {
  width: 290px;
  height: 100vh;
  background-color: #dbb56f;
  position: fixed;
  top: 56px;
  left: -250px;
  transition: left 0.3s ease-in-out;
  z-index: 1050;
  padding: 1rem;
  display: flex;
  flex-direction: column;
}
.sidebar-show {
  left: 0;
}
.menu-scroll-area {
  overflow-y: auto;
  flex-grow: 1;
  max-height: calc(100%);
  padding-right: 6px;
}
.menu-scroll-area::-webkit-scrollbar {
  width: 6px;
}
.menu-scroll-area::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}
.main-content {
  flex-grow: 1;
  transition: margin-left 0.3s ease-in-out;
}
.content-shift {
  margin-left: 250px;
}
.navbar {
  z-index: 1100;
}
.bg-sidebar {
  background-color: #dbb56f;
}
</style>
