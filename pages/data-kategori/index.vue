<template>
    <div class="container mt-4">
      <h4>Master Data</h4>
      <div class="mb-3 d-flex align-items-center justify-content-between">
        <div class="d-flex align-items-center gap-2">
          <label class="me-2">Pilih Tabel:</label>
          <select
            class="form-select w-auto"
            v-model="selectedEntity"
            @change="handleChangeEntity"
          >
            <option v-for="entity in entities" :key="entity.name" :value="entity.name">
              {{ entity.label }}
            </option>
          </select>
        </div>
        <button class="btn btn-primary" @click="openModal('tambah')">
          Tambah {{ currentLabel }}
        </button>
      </div>
      <div class="c-table p-5">
      <table class="table table-bordered">
        <thead class="table-light">
          <tr>
            <th>No</th>
            <th>Nama</th>
            <th>Kode</th>
            <th>Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in paginatedData" :key="item.id">
            <td>{{ index + 1 + (page - 1) * perPage }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.code }}</td>
            <td>
              <button
                class="btn btn-sm btn-warning me-2"
                @click="openModal('edit', item)"
              >
                Edit
              </button>
              <button class="btn btn-sm btn-danger" @click="handleDelete(item.id)">
                Hapus
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
      <nav class="d-flex justify-content-center">
        <ul class="pagination">
          <li class="page-item" :class="{ disabled: page === 1 }">
            <button class="page-link" @click="page--" :disabled="page === 1">
              Previous
            </button>
          </li>
          <li
            class="page-item"
            v-for="p in totalPages"
            :key="p"
            :class="{ active: page === p }"
          >
            <button class="page-link" @click="page = p">{{ p }}</button>
          </li>
          <li class="page-item" :class="{ disabled: page === totalPages }">
            <button class="page-link" @click="page++" :disabled="page === totalPages">
              Next
            </button>
          </li>
        </ul>
      </nav>
      <div
        v-if="showModal"
        class="modal fade show d-block"
        style="background: rgba(0, 0, 0, 0.5)"
      >
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">
                {{ modal.mode === "tambah" ? "Tambah" : "Edit" }} {{ currentLabel }}
              </h5>
              <button class="btn-close" @click="closeModal()"></button>
            </div>
            <div class="modal-body">
              <div class="mb-3">
                <label>Nama</label>
                <input v-model="form.name" type="text" class="form-control" />
              </div>
              <div class="mb-3">
                <label>Kode</label>
                <input v-model="form.code" type="text" class="form-control" />
              </div>
            </div>
            <div class="modal-footer">
              <button class="btn btn-secondary" @click="closeModal()">Batal</button>
              <button class="btn btn-primary" @click="handleSubmit()">Simpan</button>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient();

const entities = ref([
  { name: "category_product", label: "Kategori Produk", data: [] },
  { name: "color", label: "Warna", data: [] },
  { name: "size", label: "Ukuran", data: [] },
]);

const selectedEntity = ref("category_product");
const modal = ref({ mode: "tambah", id: null });
const form = ref({ name: "", code: "" });
const showModal = ref(false);
const page = ref(1);
const perPage = 10;

const currentEntity = computed(() =>
  entities.value.find((e) => e.name === selectedEntity.value)
);
const currentLabel = computed(() => currentEntity.value?.label || "");

const paginatedData = computed(() => {
  const start = (page.value - 1) * perPage;
  return currentEntity.value?.data.slice(start, start + perPage) || [];
});
const totalPages = computed(() =>
  Math.ceil((currentEntity.value?.data.length || 0) / perPage)
);

const loadData = async () => {
  for (const entity of entities.value) {
    const { data, error } = await supabase
      .from(entity.name)
      .select("*")
      .order("created_at", { ascending: false });

    if (error) {
      console.error(`Gagal load ${entity.name}:`, error.message);
      entity.data = [];
    } else {
      entity.data = data;
    }
  }
};

const handleChangeEntity = () => {
  page.value = 1;
};

const openModal = (mode, item = null) => {
  modal.value = { mode, id: item?.id || null };
  form.value = {
    name: item?.name || "",
    code: item?.code || "",
  };
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
  form.value = { name: "", code: "" };
};

const handleSubmit = async () => {
  const entity = selectedEntity.value;
  const payload = {
    name: form.value.name.trim(),
    code: form.value.code.trim(),
  };

  if (!payload.name || !payload.code) {
    alert("Nama dan Kode wajib diisi!");
    return;
  }

  let result;
  if (modal.value.mode === "tambah") {
    result = await supabase.from(entity).insert(payload);
  } else {
    result = await supabase.from(entity).update(payload).eq("id", modal.value.id);
  }

  if (result.error) {
    alert("Gagal menyimpan data: " + result.error.message);
    return;
  }

  await loadData();
  closeModal();
};

const handleDelete = async (id) => {
  if (!confirm("Yakin ingin menghapus data ini?")) return;

  const { error } = await supabase.from(selectedEntity.value).delete().eq("id", id);

  if (error) {
    alert("Gagal menghapus data: " + error.message);
    return;
  }

  await loadData();
};

onMounted(loadData);
</script>

<style scoped>
.modal-content {
  border-radius: 12px;
}
</style>
