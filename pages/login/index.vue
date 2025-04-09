<template>
    <div class="container d-flex justify-content-center align-items-center vh-100">
      <div class="card shadow-lg d-flex flex-row" style="width: 700px;">
        <div class="card-body w-50 p-4">
          <form @submit.prevent="Login">
          <div class="mb-3">
            <label  for="email" class="form-label">Email Address</label>
            <input v-model="email" type="email" id="email" class="form-control" placeholder="Email Address" />
          </div>
          <div class="mb-3">
            <label for="password" class="form-label">Password</label>
            <input  v-model="password" type="password" id="password" class="form-control" placeholder="Password" />
          </div>
          <div class="d-flex justify-content-end mb-2">
            <a href="#" class="text-decoration-none">Lupa Sandi?</a>
          </div>
          <div>
            <button type="submit" class="btn btn-primary w-100">Masuk</button>
          </div>
        </form>

        </div>
        <div class="w-50 d-flex align-items-center justify-content-center" style="background-color: #DBB56F;">
          <img src="../../assets/img/logo.jpg" alt="Logo" class="img-fluid" />
        </div>
      </div>
    </div>
  </template>
  <script setup>
 useHead({ title: 'LOGIN' });
 const supabase = useSupabaseClient();
 const email = ref('');
 const password = ref('');

 const Login = async () => {
  const { data, error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  });

  if (error) {
    alert('Password atau email Anda salah');
  } else {
    navigateTo('../');
  }
};

</script>
  