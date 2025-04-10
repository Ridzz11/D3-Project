<template>
    <div class="container mt-4">
      <h4>Kelola Toko <small class="text-muted">Kelola Toko</small></h4>

      <div class="card p-3">
        <div class="d-flex justify-content-between align-items-center">
          <h6>Profil Toko</h6>
          <button class="btn btn-warning" @click="bukaModal">Edit Profil</button>
        </div>
        <table class="table table-bordered mt-2" v-if="profil">
          <tbody>
            <tr>
              <th>Nama Toko</th>
              <td>{{ profil.name }}</td>
            </tr>
            <tr>
              <th>Pemilik</th>
              <td>{{ profil.pemilik }}</td>
            </tr>
            <tr>
              <th>Alamat</th>
              <td>{{ profil.alamat }}</td>
            </tr>
            <tr>
              <th>Email</th>
              <td>{{ profil.email }}</td>
            </tr>
            <tr>
              <th>No. HP</th>
              <td>{{ profil.no_hp }}</td>
            </tr>
            <tr>
              <th>Media Sosial</th>
              <td>{{ profil.media_sosial }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div v-if="showModal" class="modal fade show d-block" style="background: rgba(0, 0, 0, 0.5);">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Edit Profil Toko</h5>
            <button type="button" class="btn-close" @click="showModal = false"></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="simpanProfil">
              <div class="mb-3">
                <label class="form-label">Nama Toko</label>
                <input v-model="form.name" type="text" class="form-control" />
              </div>
              <div class="mb-3">
                <label class="form-label">Pemilik</label>
                <input v-model="form.pemilik" type="text" class="form-control" />
              </div>
              <div class="mb-3">
                <label class="form-label">Alamat</label>
                <input v-model="form.alamat" type="text" class="form-control" />
              </div>
              <div class="mb-3">
                <label class="form-label">Email</label>
                <input v-model="form.email" type="email" class="form-control" />
              </div>
              <div class="mb-3">
                <label class="form-label">No. HP</label>
                <input v-model="form.no_hp" type="text" class="form-control" />
              </div>
              <div class="mb-3">
                <label class="form-label">Media Sosial</label>
                <input v-model="form.media_sosial" type="text" class="form-control" />
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-danger" @click="showModal = false">Batal</button>
            <button type="button" class="btn btn-success" @click="simpanProfil">Simpan</button>
          </div>
        </div>
      </div>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient();
const route = useRoute();

const showModal = ref(false)
const profil = ref(null)
const form = ref({
  name: '',
  pemilik: '',
  alamat: '',
  email: '',
  no_hp: '',
  media_sosial: '',
})

const getProfil = async () => {
  const { data, error } = await supabase.from('profile_toko').select('*').single()
  if (error) {
    console.error(error)
  } else {
    profil.value = data
  }
}

const bukaModal = () => {
  form.value = { ...profil.value }
  showModal.value = true
}

const simpanProfil = async () => {
  const { error } = await supabase
    .from('profile_toko')
    .update({ ...form.value })
    .eq('id', profil.value.id)

  if (error) {
    alert('Gagal menyimpan profil!')
    console.error(error)
  } else {
    showModal.value = false
    getProfil()
    alert('Profil berhasil diperbarui!')
  }
}

onMounted(() => {
  getProfil()
})
</script>

<style scoped>
.modal-lg {
  max-width: 800px;
}
</style>
