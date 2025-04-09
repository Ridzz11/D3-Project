<template>
  <Header>
    <div class="container mt-4">
      <h4>
        Data Barang
        <small class="text-muted">Data Barang</small>
      </h4>

      <button class="btn btn-primary mb-3" @click="tambahBarang">
        + Tambah Barang
      </button>

      <table class="table table-bordered">
        <thead>
          <tr>
            <th>No</th>
            <th>Kode Barang</th>
            <th>Nama Barang</th>
            <th>Harga Barang</th>
            <th>Warna</th>
            <th>Ukuran</th>
            <th>Kategori</th>
            <th>Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in paginatedProducts" :key="item.id">
            <td>{{ (currentPage - 1) * itemsPerPage + index + 1 }}</td>
            <td>{{ item.code }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.price }}</td>
            <td>{{ item.color_name }}</td>
            <td>{{ item.size_name }}</td>
            <td>{{ item.category_name }}</td>
            <td>
              <button class="btn btn-success btn-sm me-1" @click="editBarang(item)">Edit</button>
              <button class="btn btn-danger btn-sm" @click="konfirmasiHapus(item.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
      <nav class="d-flex justify-content-between align-items-center">
        <small>Halaman {{ currentPage }} dari {{ totalPages }}</small>
        <div>
          <button class="btn btn-outline-primary me-2" :disabled="currentPage === 1" @click="prevPage">
            Previous
          </button>
          <button class="btn btn-outline-primary" :disabled="currentPage === totalPages" @click="nextPage">
            Next
          </button>
        </div>
      </nav>
    </div>
  </Header>
</template>

<script setup>
const supabase = useSupabaseClient();

const name = ref('');
const price = ref(0);
const color = ref(null);
const size = ref(null);
const category = ref(null);
const showModal = ref(false);
const isEdit = ref(false);
const showDeleteModal = ref(false);
const confirmDeleteId = ref(null);

const products = ref([]);
const colors = ref([]);
const sizes = ref([]);
const categories = ref([]);

const currentPage = ref(1);
const itemsPerPage = ref(5);

const paginatedProducts = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  return products.value.slice(start, start + itemsPerPage.value);
});

const totalPages = computed(() => {
  return Math.ceil(products.value.length / itemsPerPage.value);
});

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++;
};

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--;
};

const tambahBarang = () => {
  resetForm();
  showModal.value = true;
  isEdit.value = false;
};

const resetForm = () => {
  name.value = '';
  price.value = 0;
  color.value = null;
  size.value = null;
  category.value = null;
};

const getOptions = async () => {
  const { data: colorData } = await supabase.from('color').select('id, name, code');
  const { data: sizeData } = await supabase.from('size').select('id, name, code');
  const { data: categoryData } = await supabase.from('category_product').select('id, name, code');

  colors.value = colorData || [];
  sizes.value = sizeData || [];
  categories.value = categoryData || [];
};

const getProducts = async () => {
  const { data } = await supabase
    .from('products')
    .select(`
      id,
      name,
      code,
      price,
      color:color(id, name, code),
      size:size(id, name, code),
      category:category_product(id, name, code)
    `)
    .order('created_at', { ascending: false }); 

  products.value = (data || []).map(item => ({
    ...item,
    color_name: item.color?.name,
    size_name: item.size?.name,
    category_name: item.category?.name
  }));

  currentPage.value = 1; 
};

const Product = async () => {
  const totalData = await supabase.from('products').select('id', { count: 'exact' });
  const nextIndex = (totalData.count || 0) + 1;

  const categoryCode = categories.value.find(i => i.id === category.value)?.code || 'XX';
  const colorCode = colors.value.find(i => i.id === color.value)?.code || 'YY';
  const sizeCode = sizes.value.find(i => i.id === size.value)?.code || 'ZZ';

  const prefix = name.value.trim().charAt(0).toUpperCase();
  const code = `${categoryCode}-${prefix}${nextIndex}${colorCode}-${sizeCode}`;

  const { error } = await supabase.from('products').insert([{
    name: name.value,
    price: price.value,
    color_id: color.value,
    category_id: category.value,
    size_id: size.value,
    code
  }]);

  if (!error) {
    alert('Data berhasil ditambahkan!');
    getProducts();
    tutupModal();
  } else {
    console.error('Error inserting data:', error);
  }
};

const tutupModal = () => {
  showModal.value = false;
  resetForm();
};

const konfirmasiHapus = (id) => {
  confirmDeleteId.value = id;
  showDeleteModal.value = true;
};

const deleteBarang = async (id) => {
  const { error } = await supabase.from('products').delete().eq('id', id);
  if (!error) {
    getProducts();
    showDeleteModal.value = false;
  } else {
    console.error('Gagal menghapus:', error);
  }
};

const editBarang = (item) => {
  name.value = item.name;
  price.value = item.price;
  color.value = item.color?.id;
  size.value = item.size?.id;
  category.value = item.category?.id;
  showModal.value = true;
  isEdit.value = true;
};

onMounted(() => {
  getOptions();
  getProducts();
});
</script>

<style scoped>
.modal-lg {
  max-width: 800px;
}
</style>
