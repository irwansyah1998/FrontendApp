<template>
    <div class="crud-page">
      <h1>Manajemen Produk</h1>
      
      <!-- Form Tambah/Edit -->
      <form @submit.prevent="isEditing ? updateItem() : addItem()">
        <input v-model="formItem.name" placeholder="Nama Produk" required />
        <input v-model.number="formItem.price" placeholder="Harga Produk" type="number" required />
        <input v-model="formItem.description" placeholder="Deskripsi" />
        <button type="submit">{{ isEditing ? 'Update' : 'Tambah' }}</button>
        <button v-if="isEditing" @click="cancelEdit">Batal</button>
      </form>
  
      <!-- Daftar Produk -->
      <ul>
        <li v-for="(product, index) in products" :key="product.id">
          <span>{{ product.name }} - {{ product.price | currency }}</span>
          <p>{{ product.description }}</p>
          <button @click="editItem(product, index)">Edit</button>
          <button @click="removeItem(product.id)">Hapus</button>
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        products: [], // Menyimpan daftar produk dari API
        formItem: { name: '', price: 0, description: '' },
        isEditing: false,
        currentIndex: null,
        currentId: null
      };
    },
    methods: {
      // Ambil produk dari API saat komponen di-mount
      async fetchProducts() {
        try {
          const response = await axios.get('http://backendapp.test/api/products');
          this.products = response.data;
        } catch (error) {
          console.error('Gagal mengambil data produk:', error);
        }
      },
      addItem() {
        if (this.formItem.name && this.formItem.price) {
          const newProduct = { ...this.formItem };
          this.products.push(newProduct);
          this.resetForm();
        }
      },
      editItem(product, index) {
        this.isEditing = true;
        this.currentIndex = index;
        this.formItem = { ...product };
        this.currentId = product.id;
      },
      updateItem() {
        if (this.currentId !== null) {
          const index = this.products.findIndex(p => p.id === this.currentId);
          if (index !== -1) {
            this.products[index] = { ...this.formItem, id: this.currentId };
            this.resetForm();
          }
        }
      },
      cancelEdit() {
        this.resetForm();
      },
      removeItem(id) {
        this.products = this.products.filter(product => product.id !== id);
      },
      resetForm() {
        this.formItem = { name: '', price: 0, description: '' };
        this.isEditing = false;
        this.currentIndex = null;
        this.currentId = null;
      }
    },
    filters: {
      currency(value) {
        return `Rp${value.toFixed(2)}`;
      }
    },
    mounted() {
      this.fetchProducts();
    }
  };
  </script>
  
  <style scoped>
  .crud-page {
    width: 300px;
    margin: 0 auto;
  }
  form {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
    flex-direction: column;
  }
  input {
    padding: 5px;
  }
  button {
    padding: 5px 10px;
    color: white;
    background-color: #42b983;
    border: none;
    cursor: pointer;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    margin: 10px 0;
    padding: 5px;
    border: 1px solid #ccc;
  }
  </style>
  