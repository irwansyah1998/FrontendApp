<template>
  <div id="app">
    <!-- Login menjadi gerbang sebelum halaman produk dapat diakses. -->
    <LoginPage v-if="!authToken" @authenticated="handleAuthenticated" />
    <CrudPage v-else :auth-token="authToken" @logout="logout" @unauthorized="handleUnauthorized" />
  </div>
</template>

<script>
import CrudPage from './components/CrudPage.vue'
import LoginPage from './components/LoginPage.vue'

export default {
  name: 'App',
  components: {
    CrudPage,
    LoginPage
  },
  data() {
    return {
      // Memulihkan sesi browser agar pengguna tidak perlu login setiap refresh.
      authToken: this.readStoredToken()
    };
  },
  methods: {
    readStoredToken() {
      try {
        return localStorage.getItem('backendapp_api_token') || '';
      } catch (error) {
        // Mode privasi/browser policy dapat menolak storage; aplikasi tetap harus bisa dibuka.
        return '';
      }
    },
    handleAuthenticated(token) {
      this.authToken = token;
    },
    logout() {
      try {
        localStorage.removeItem('backendapp_api_token');
      } catch (error) {
        // Token in-memory tetap dihapus di bawah meskipun storage tidak tersedia.
      }
      this.authToken = '';
    },
    handleUnauthorized() {
      this.logout();
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0;
}
</style>
