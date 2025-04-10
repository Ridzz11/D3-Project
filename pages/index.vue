<template>
    <div class="container mt-4">
      <h4>Dashboard <small class="text-muted">Control Panel</small></h4>
      <div class="row my-4">
        <div class="col-md-4">
          <div class="card text-white bg-primary p-3">
            <h5>{{ totalProduct }}</h5>
            <p><i class="bi bi-box"></i> Total Product</p>
          </div>
        </div>
        <div class="col-md-4">
          <div class="card text-white bg-success p-3">
            <h5>{{ totalQty }}</h5>
            <p><i class="bi bi-box-seam"></i> Total Qty</p>
          </div>
        </div>
        <div class="col-md-4">
          <div class="card text-white bg-warning p-3">
            <h5>Rp {{ totalPrice.toLocaleString() }}</h5>
            <p><i class="bi bi-cash-coin"></i> Total Harga</p>
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-md-6">
          <div class="card p-3">
            <h6>Barang Masuk Bulan Ini</h6>
            <div class="table-responsive">
              <table class="table table-bordered mt-2">
                <thead>
                  <tr>
                    <th>Kode Barang</th>
                    <th>Nama</th>
                    <th>Tanggal</th>
                    <th>Jumlah</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in barangMasuk" :key="item.code">
                    <td>{{ item.code }}</td>
                    <td>{{ item.name }}</td>
                    <td>{{ new Date(item.created_at).toLocaleDateString() }}</td>
                    <td>{{ item.qty_in }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>

        <div class="col-md-6">
          <div class="card p-3">
            <h6>Barang Keluar Bulan Ini</h6>
            <div class="table-responsive">
              <table class="table table-bordered mt-2">
                <thead>
                  <tr>
                    <th>Kode Barang</th>
                    <th>Nama</th>
                    <th>Tanggal</th>
                    <th>Jumlah</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in barangKeluar" :key="item.code">
                    <td>{{ item.code }}</td>
                    <td>{{ item.name }}</td>
                    <td>{{ new Date(item.created_at).toLocaleDateString() }}</td>
                    <td>{{ item.qty_out }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()

const totalProduct = ref(0)
const totalQty = ref(0)
const totalPrice = ref(0)
const barangMasuk = ref([])
const barangKeluar = ref([])

const fetchDashboardData = async () => {
  const { count: productCount } = await supabase
    .from('products')
    .select('*', { count: 'exact', head: true })
  totalProduct.value = productCount ?? 0

  const { data: stockData } = await supabase
    .from('stocks')
    .select('qty_end, price_end')

  totalQty.value = stockData?.reduce((sum, item) => sum + item.qty_end, 0) ?? 0
  totalPrice.value = stockData?.reduce((sum, item) => sum + item.price_end, 0) ?? 0

  const { data: masukData } = await supabase
    .from('mutation_stock')
    .select('code, name, created_at, qty_in')
    .gt('qty_in', 0)
    .order('created_at', { ascending: false })
    .limit(5)
  barangMasuk.value = masukData ?? []

  const { data: keluarData } = await supabase
    .from('mutation_stock')
    .select('code, name, created_at, qty_out')
    .gt('qty_out', 0)
    .order('created_at', { ascending: false })
    .limit(5)
  barangKeluar.value = keluarData ?? []
}

onMounted(() => {
  fetchDashboardData()
})
</script>
