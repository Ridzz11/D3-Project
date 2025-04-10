<template>
    <div class="container mt-4">
      <h4>
        Monitoring Barang
        <small class="text-muted">Mutasi Stok</small>
      </h4>
      <div class="row g-3 mb-4">
        <div class="col-md-6">
          <div class="card text-white bg-primary">
            <div class="card-body">
              <h5 class="card-title">Total Qty Akhir</h5>
              <p class="card-text fs-4">{{ totalQtyEnd }}</p>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="card text-white bg-success">
            <div class="card-body">
              <h5 class="card-title">Total Harga Akhir</h5>
              <p class="card-text fs-4">{{ formatCurrency(totalPriceEnd) }}</p>
            </div>
          </div>
        </div>
      </div>
      <div class="row g-2 mb-2">
        <div class="col-md-6">
          <input v-model="filters.code" type="text" class="form-control" placeholder="Kode Barang" />
        </div>
        <div class="col-md-6">
          <input v-model="filters.name" type="text" class="form-control" placeholder="Nama Barang" />
        </div>
      </div>
      <div class="row g-2 mb-2">
        <div class="col-md-6">
          <input v-model="filters.start_date" type="date" class="form-control" />
        </div>
        <div class="col-md-6">
          <input v-model="filters.end_date" type="date" class="form-control" />
        </div>
      </div>
      <div class="mb-3">
        <button class="btn btn-primary me-2" @click="applyFilter">Filter</button>
        <button class="btn btn-secondary" @click="clearFilter">Clear</button>
      </div>
      <table class="table table-bordered table-hover">
        <thead class="table-light">
          <tr>
            <th>No</th>
            <th>Kode</th>
            <th>Nama</th>
            <th>Qty In</th>
            <th>Qty Out</th>
            <th>Qty End</th>
            <th>Harga In</th>
            <th>Harga Out</th>
            <th>Harga Akhir</th>
            <th>Tanggal</th>
            <th>Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in paginatedData" :key="item.id">
            <td>{{ index + 1 + (currentPage - 1) * perPage }}</td>
            <td>{{ item.code }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.qty_in }}</td>
            <td>{{ item.qty_out }}</td>
            <td>{{ item.qty_end }}</td>
            <td>{{ formatCurrency(item.price_in) }}</td>
            <td>{{ formatCurrency(item.price_out) }}</td>
            <td>{{ formatCurrency(item.price_end) }}</td>
            <td>{{ item.created_at?.split("T")[0] }}</td>
            <td>
              <button class="btn btn-sm btn-warning me-1" @click="editData(item)">
                <i class="bi bi-pencil"></i>
              </button>
              <button class="btn btn-sm btn-danger" @click="confirmDelete(item)">
                <i class="bi bi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <nav>
        <ul class="pagination justify-content-center">
          <li class="page-item" :class="{ disabled: currentPage === 1 }">
            <button class="page-link" @click="currentPage--" :disabled="currentPage === 1">Previous</button>
          </li>
          <li
            class="page-item"
            v-for="page in totalPages"
            :key="page"
            :class="{ active: currentPage === page }"
          >
            <button class="page-link" @click="currentPage = page">{{ page }}</button>
          </li>
          <li class="page-item" :class="{ disabled: currentPage === totalPages }">
            <button class="page-link" @click="currentPage++" :disabled="currentPage === totalPages">Next</button>
          </li>
        </ul>
      </nav>
    </div>

    <div v-if="showModal" class="modal fade show d-block" style="background: rgba(0, 0, 0, 0.5)">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Konfirmasi</h5>
            <button type="button" class="btn-close" @click="closeModal"></button>
          </div>
          <div class="modal-body">
            Apakah kamu yakin ingin <strong>{{ modalMode }}</strong> data <strong>{{ selectedItem.name }}</strong>?
          </div>
          <div class="modal-footer">
            <button class="btn btn-secondary" @click="closeModal">Batal</button>
            <button class="btn btn-danger" @click="handleModalAction">Ya, Lanjut</button>
          </div>
        </div>
      </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient();
const route = useRoute();

const monitoringData = ref([]);
const filters = ref({
  code: "",
  name: "",
  start_date: "",
  end_date: "",
});

const showModal = ref(false);
const modalMode = ref("");
const selectedItem = ref({});

const totalQtyEnd = ref(0);
const totalPriceEnd = ref(0);

const currentPage = ref(1);
const perPage = 10;

const locationId = computed(() => {
  const id = Number(route.params?.id || 2);
  return isNaN(id) ? 2 : id;
});

const getMonitoringData = async () => {
  const { data } = await supabase
    .from("mutation_stock")
    .select("*")
    .eq("location_id", locationId.value)
    .order("created_at", { ascending: false });

  monitoringData.value = data || [];
};

const getTotalFromStocks = async () => {
  const { data } = await supabase
    .from("stocks")
    .select("qty_end, price_end")
    .eq("location_id", locationId.value);

  if (data && Array.isArray(data)) {
    totalQtyEnd.value = data.reduce((sum, item) => sum + (item.qty_end || 0), 0);
    totalPriceEnd.value = data.reduce((sum, item) => sum + (item.price_end || 0), 0);
  }
};

const formatCurrency = (val) => {
  return "Rp " + (val || 0).toLocaleString("id-ID");
};

const filteredData = computed(() => {
  return monitoringData.value.filter((item) => {
    const matchCode = filters.value.code
      ? item.code?.toLowerCase().includes(filters.value.code.toLowerCase())
      : true;

    const matchName = filters.value.name
      ? item.name?.toLowerCase().includes(filters.value.name.toLowerCase())
      : true;

    const date = item.created_at?.split("T")[0];
    const matchStart = filters.value.start_date ? date >= filters.value.start_date : true;
    const matchEnd = filters.value.end_date ? date <= filters.value.end_date : true;

    return matchCode && matchName && matchStart && matchEnd;
  });
});

const totalPages = computed(() => {
  return Math.ceil(filteredData.value.length / perPage);
});

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * perPage;
  const end = start + perPage;
  return filteredData.value.slice(start, end);
});

const applyFilter = () => {
  currentPage.value = 1;
};

const clearFilter = () => {
  filters.value = {
    code: "",
    name: "",
    start_date: "",
    end_date: "",
  };
  currentPage.value = 1;
};

const confirmDelete = (item) => {
  selectedItem.value = item;
  modalMode.value = "hapus";
  showModal.value = true;
};

const editData = (item) => {
  selectedItem.value = item;
  modalMode.value = "edit";
  showModal.value = true;
};

const handleModalAction = async () => {
  if (modalMode.value === "hapus") {
    await supabase.from("mutation_stock").delete().eq("id", selectedItem.value.id);
  }
  showModal.value = false;
  getMonitoringData();
  getTotalFromStocks();
};

const closeModal = () => {
  showModal.value = false;
};

onMounted(() => {
  getMonitoringData();
  getTotalFromStocks();
});
</script>

<style scoped>
.table th,
.table td {
  vertical-align: middle;
}
.modal-content {
  border-radius: 12px;
}
</style>