<template>
    <div class="container mt-4">
      <h4>
        Barang Keluar
        <small class="text-muted">Barang Keluar</small>
      </h4>
      <button class="btn btn-primary mb-3" @click="tambahBarangKeluar">
        + Barang Keluar
      </button>
      <table class="table table-bordered">
        <thead>
          <tr>
            <th>No</th>
            <th>Kode Barang</th>
            <th>Nama Barang</th>
            <th>Tanggal</th>
            <th>Qty In</th>
            <th>Qty Out</th>
            <th>Qty End</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in barangKeluar" :key="item.id">
            <td>{{ index + 1 + (currentPage - 1) * itemsPerPage }}</td>
            <td>{{ item.products?.code }}</td>
            <td>{{ item.products?.name }}</td>
            <td>{{ item.created_at?.split('T')[0] }}</td>
            <td>{{ item.qty_in }}</td>
            <td>{{ item.qty_out }}</td>
            <td>{{ item.qty_end }}</td>
          </tr>
        </tbody>
      </table>

      <nav aria-label="Page navigation">
        <ul class="pagination justify-content-end">
          <li class="page-item" :class="{ disabled: currentPage === 1 }">
            <button class="page-link" @click="changePage(currentPage - 1)">Previous</button>
          </li>
          <li
            v-for="page in totalPages"
            :key="page"
            class="page-item"
            :class="{ active: currentPage === page }"
          >
            <button class="page-link" @click="changePage(page)">{{ page }}</button>
          </li>
          <li class="page-item" :class="{ disabled: currentPage === totalPages }">
            <button class="page-link" @click="changePage(currentPage + 1)">Next</button>
          </li>
        </ul>
      </nav>
    </div>

    <div v-if="showModal" class="modal fade show d-block" style="background: rgba(0, 0, 0, 0.5);">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Barang Keluar</h5>
            <button type="button" class="btn-close" @click="tutupModal"></button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label">Pilih Produk</label>
              <select v-model.number="form.product_id" class="form-select">
                <option disabled value="">Pilih Produk</option>
                <option v-for="item in products" :key="item.id" :value="item.id">
                  {{ item.name }} - {{ item.code }}
                </option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label">Qty Out</label>
              <input v-model.number="form.qty_out" type="number" class="form-control" />
            </div>
          </div>
          <div class="modal-footer">
            <button class="btn btn-danger" @click="tutupModal">Batal</button>
            <button class="btn btn-success" @click="submitBarangKeluar">Simpan</button>
          </div>
        </div>
      </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient();

const showModal = ref(false);
const products = ref([]);
const barangKeluar = ref([]);

const form = ref({
  product_id: null,
  qty_out: null,
});

const currentPage = ref(1);
const itemsPerPage = 20;
const totalItems = ref(0);

const totalPages = computed(() => {
  return Math.ceil(totalItems.value / itemsPerPage);
});

const changePage = (page) => {
  if (page < 1 || page > totalPages.value) return;
  currentPage.value = page;
  getBarangKeluar();
};

const getProducts = async () => {
  const { data } = await supabase.from('products').select('id, name, price, code');
  products.value = data || [];
};

const getBarangKeluar = async () => {
  const from = (currentPage.value - 1) * itemsPerPage;
  const to = from + itemsPerPage - 1;

  const { data, count } = await supabase
    .from('stocks')
    .select(`
      id,
      product_id,
      qty_in,
      qty_out,
      qty_end,
      price_in,
      price_out,
      price_end,
      created_at,
      products (
        id,
        name,
        code
      )
    `, { count: 'exact' })
    .eq('location_id', 1)
    .order('id', { ascending: false })
    .range(from, to);

  barangKeluar.value = data || [];
  totalItems.value = count || 0;
};

const tambahBarangKeluar = () => {
  resetForm();
  showModal.value = true;
};

const resetForm = () => {
  form.value = {
    product_id: null,
    qty_out: null,
  };
};

const tutupModal = () => {
  showModal.value = false;
  resetForm();
};

const submitBarangKeluar = async () => {
  const productId = form.value.product_id;
  const inputQtyOut = form.value.qty_out || 0;

  const product = products.value.find(p => p.id === productId);
  if (!product) return;

  const productPrice = product.price || 0;
  const inputPriceOut = inputQtyOut * productPrice;

  const { data: existingData } = await supabase
    .from('stocks')
    .select('id, qty_in, qty_out, qty_end, price_in, price_out, price_end')
    .eq('product_id', productId)
    .eq('location_id', 1)
    .limit(1);

  if (!existingData || existingData.length === 0) {
    alert("Data stok belum tersedia untuk produk ini.");
    return;
  }

  const existing = existingData[0];

  const newQtyOut = (existing.qty_out || 0) + inputQtyOut;
  const newQtyEnd = (existing.qty_in || 0) - newQtyOut;
  const newPriceOut = (existing.price_out || 0) + inputPriceOut;
  const newPriceEnd = (existing.price_in || 0) - newPriceOut;

  await supabase
    .from('stocks')
    .update({
      qty_out: newQtyOut,
      qty_end: newQtyEnd,
      price_out: newPriceOut,
      price_end: newPriceEnd,
      updated_at: new Date().toISOString(),
    })
    .eq('id', existing.id);

  const { data: finalStock } = await supabase
    .from('stocks')
    .select('*')
    .eq('product_id', productId)
    .eq('location_id', 1)
    .limit(1)
    .single();

  if (!finalStock) return;

  await supabase.from('mutation_stock').insert([{
    name: product.name,
    code: product.code,
    qty_in: 0,
    qty_out: inputQtyOut,
    qty_end: finalStock.qty_end,
    price_in: 0,
    price_out: inputPriceOut,
    price_end: finalStock.price_end,
    location_id: 1,
    created_at: new Date().toISOString(),
  }]);

  getBarangKeluar();
  tutupModal();
};

onMounted(() => {
  getProducts();
  getBarangKeluar();
});
</script>

<style scoped>
.modal-lg {
  max-width: 800px;
}
</style>
