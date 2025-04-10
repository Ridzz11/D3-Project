<template>
    <div class="container mt-4">
      <h4>
        Barang Masuk
        <small class="text-muted">Barang Masuk</small>
      </h4>
      <button class="btn btn-primary mb-3" @click="tambahBarangMasuk">
        + Tambah Barang
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
          <tr v-for="(item, index) in barangMasuk" :key="item.id">
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
            <h5 class="modal-title">Tambah Barang Masuk</h5>
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
              <label class="form-label">Qty In</label>
              <input v-model.number="form.qty_in" type="number" class="form-control" />
            </div>
          </div>
          <div class="modal-footer">
            <button class="btn btn-danger" @click="tutupModal">Batal</button>
            <button class="btn btn-success" @click="submitBarangMasuk">Simpan</button>
          </div>
        </div>
      </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient();

const showModal = ref(false);
const products = ref([]);
const barangMasuk = ref([]);

const form = ref({
  product_id: null,
  qty_in: null,
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
  getBarangMasuk();
};

const getProducts = async () => {
  const { data } = await supabase.from('products').select('id, name, price, code');
  products.value = data || [];
};

const getBarangMasuk = async () => {
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

  barangMasuk.value = data || [];
  totalItems.value = count || 0;
};

const tambahBarangMasuk = () => {
  resetForm();
  showModal.value = true;
};

const resetForm = () => {
  form.value = {
    product_id: null,
    qty_in: null,
  };
};

const tutupModal = () => {
  showModal.value = false;
  resetForm();
};

const submitBarangMasuk = async () => {
  const productId = form.value.product_id;
  const inputQtyIn = form.value.qty_in || 0;

  const product = products.value.find(p => p.id === productId);
  if (!product) return;

  const productPrice = product.price || 0;
  const inputPriceIn = inputQtyIn * productPrice;

  const { data: existingData } = await supabase
    .from('stocks')
    .select('id, qty_in, qty_out, qty_end, price_in, price_out, price_end')
    .eq('product_id', productId)
    .eq('location_id', 1)
    .limit(1);

  if (existingData && existingData.length > 0) {
    const existing = existingData[0];
    const newQtyIn = (existing.qty_in || 0) + inputQtyIn;
    const newQtyEnd = newQtyIn - (existing.qty_out || 0);
    const newPriceIn = (existing.price_in || 0) + inputPriceIn;
    const newPriceEnd = newPriceIn - (existing.price_out || 0);

    await supabase
      .from('stocks')
      .update({
        qty_in: newQtyIn,
        qty_end: newQtyEnd,
        price_in: newPriceIn,
        price_end: newPriceEnd,
        updated_at: new Date().toISOString(),
      })
      .eq('id', existing.id);
  } else {
    await supabase.from('stocks').insert([{
      product_id: productId,
      qty_in: inputQtyIn,
      qty_out: 0,
      qty_end: inputQtyIn,
      price_in: inputPriceIn,
      price_out: 0,
      price_end: inputPriceIn,
      location_id: 1,
      updated_at: new Date().toISOString(),
    }]);
  }

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
    qty_in: inputQtyIn,
    qty_out: 0,
    qty_end: finalStock.qty_end,
    price_in: inputPriceIn,
    price_out: 0,
    price_end: finalStock.price_end,
    location_id: 1,
    created_at: new Date().toISOString(),
  }]);

  getBarangMasuk();
  tutupModal();
};

onMounted(() => {
  getProducts();
  getBarangMasuk();
});
</script>

<style scoped>
.modal-lg {
  max-width: 800px;
}
</style>
