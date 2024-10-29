<template>
    <div class="crud-page">
        <h1>Manajemen Produk</h1>

        <!-- Form Tambah/Edit -->
        <form @submit.prevent="isEditing ? updateItem() : addItem()" class="crud-form">
            <input v-model="formItem.name" placeholder="Nama Produk" required />
            <input v-model.number="formItem.price" placeholder="Harga Produk" type="number" required />
            <input v-model="formItem.description" placeholder="Deskripsi" />
            <button type="submit">{{ isEditing ? 'Update' : 'Tambah' }}</button>
            <button v-if="isEditing" @click="cancelEdit">Batal</button>
        </form>

        <!-- Tabel Produk -->
        <table class="product-table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Nama Produk</th>
                    <th>Harga</th>
                    <th>Deskripsi</th>
                    <th>Aksi</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="product in products" :key="product.id">
                    <td>{{ product.id }}</td>
                    <td>{{ product.name }}</td>
                    <td>Rp{{ product.price.toFixed(2) }}</td>
                    <td>{{ product.description }}</td>
                    <td>
                        <button @click="editItem(product)">Edit</button>
                        <button @click="removeItem(product.id)">Hapus</button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            products: [],
            formItem: { name: '', price: 0, description: '' },
            isEditing: false,
            currentId: null
        };
    },
    methods: {
        async fetchProducts() {
            try {
                const response = await axios.get(`${process.env.VUE_APP_BACKEND}/api/products`);
                this.products = response.data.map(product => ({
                    ...product,
                    price: parseFloat(product.price) // Konversi ke angka
                }));
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
        editItem(product) {
            this.isEditing = true;
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
            this.currentId = null;
        }
    },
    mounted() {
        this.fetchProducts();
    }
};
</script>

<style scoped>
.crud-page {
    width: 80%;
    margin: 0 auto;
}

.crud-form {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

input {
    padding: 5px;
    width: 100%;
}

button {
    padding: 5px 10px;
    color: white;
    background-color: #42b983;
    border: none;
    cursor: pointer;
}

.product-table {
    width: 100%;
    border-collapse: collapse;
}

.product-table th,
.product-table td {
    padding: 8px;
    border: 1px solid #ddd;
    text-align: left;
}

.product-table th {
    background-color: #f2f2f2;
}

.product-table td button {
    margin-right: 5px;
}
</style>