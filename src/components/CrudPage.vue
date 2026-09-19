<template>
    <div class="crud-page">
        <!-- Struktur dashboard baru hanya mengubah presentasi, bukan event atau state CRUD. -->
        <header class="page-header">
            <div>
                <p class="eyebrow">Product workspace</p>
                <h1>Manajemen Produk</h1>
                <p class="page-subtitle">Kelola katalog produk Anda dari satu tempat.</p>
            </div>
            <button type="button" class="button button-ghost" @click="$emit('logout')">Keluar</button>
        </header>

        <main class="dashboard-content">
            <section class="summary-grid" aria-label="Ringkasan produk">
                <article class="summary-card summary-card-accent">
                    <span class="summary-label">Total produk</span>
                    <strong>{{ products.length }}</strong>
                    <span class="summary-note">Item dalam katalog</span>
                </article>
                <article class="summary-card">
                    <span class="summary-label">Status sistem</span>
                    <strong class="system-status"><span class="status-dot"></span>Terhubung</strong>
                    <span class="summary-note">Sinkron dengan API</span>
                </article>
            </section>

            <section class="editor-section" aria-labelledby="editor-title">
                <div class="section-heading">
                    <div>
                        <p class="section-kicker">Catalog editor</p>
                        <h2 id="editor-title">{{ isEditing ? 'Edit produk' : 'Tambah produk baru' }}</h2>
                    </div>
                    <span class="form-state">{{ isEditing ? 'Mode edit' : 'Draft baru' }}</span>
                </div>

                <form @submit.prevent="isEditing ? updateItem() : addItem()" class="crud-form">
                    <div class="field-group">
                        <label for="product-name">Nama produk</label>
                        <input id="product-name" v-model.trim="formItem.name" placeholder="Contoh: Paket Starter" required />
                    </div>
                    <div class="field-group">
                        <label for="product-price">Harga</label>
                        <div class="input-prefix">
                            <span>Rp</span>
                            <input id="product-price" v-model.number="formItem.price" placeholder="0" type="number" min="0.01" step="0.01" required />
                        </div>
                    </div>
                    <div class="field-group field-description">
                        <label for="product-description">Deskripsi <span>(opsional)</span></label>
                        <input id="product-description" v-model="formItem.description" placeholder="Tambahkan ringkasan singkat" />
                    </div>
                    <div class="form-actions">
                        <button type="submit" class="button button-primary" :disabled="isLoading">
                            {{ isEditing ? 'Simpan perubahan' : 'Tambah produk' }}
                        </button>
                        <button v-if="isEditing" type="button" class="button button-secondary" @click="cancelEdit">Batal</button>
                    </div>
                </form>
            </section>

            <!-- Status API dibuat sebagai alert agar mudah dipindai dan tetap accessible. -->
            <p v-if="isLoading" class="status-message loading-message" role="status"><span class="loading-indicator"></span>Memuat data...</p>
            <p v-if="errorMessage" class="status-message error-message" role="alert">{{ errorMessage }}</p>
            <p v-if="successMessage" class="status-message success-message" role="status">{{ successMessage }}</p>

            <section class="table-section" aria-labelledby="catalog-title">
                <div class="section-heading table-heading">
                    <div>
                        <p class="section-kicker">Live catalog</p>
                        <h2 id="catalog-title">Daftar produk</h2>
                    </div>
                    <span class="record-count">{{ products.length }} produk</span>
                </div>

                <div v-if="products.length" class="table-wrapper">
                    <table class="product-table">
                        <thead>
                            <tr>
                                <th scope="col">ID</th>
                                <th scope="col">Produk</th>
                                <th scope="col">Harga</th>
                                <th scope="col">Deskripsi</th>
                                <th scope="col"><span class="sr-only">Aksi</span></th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="product in products" :key="product.id">
                                <td class="product-id">#{{ product.id }}</td>
                                <td class="product-name">{{ product.name }}</td>
                                <td class="product-price">Rp{{ product.price.toFixed(2) }}</td>
                                <td class="product-description">{{ product.description || '-' }}</td>
                                <td class="product-actions">
                                    <button type="button" class="action-button" :disabled="isLoading" @click="editItem(product)">Edit</button>
                                    <button type="button" class="action-button action-danger" :disabled="isLoading" @click="removeItem(product.id)">Hapus</button>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
                <div v-else-if="!isLoading" class="empty-state">
                    <span class="empty-icon">+</span>
                    <h3>Belum ada produk</h3>
                    <p>Tambahkan produk pertama Anda menggunakan form di atas.</p>
                </div>
            </section>
        </main>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    props: {
        authToken: {
            type: String,
            default: ''
        }
    },
    data() {
        return {
            products: [],
            formItem: { name: '', price: 0, description: '' },
            isEditing: false,
            currentId: null,
            isLoading: false,
            errorMessage: '',
            successMessage: ''
        };
    },
    methods: {
        // Menyatukan konfigurasi request agar token tidak pernah ditulis di source code.
        getRequestConfig() {
            const token = this.authToken;

            return {
                headers: {
                    Accept: 'application/json',
                    ...(token ? { Authorization: `Bearer ${token}` } : {})
                }
            };
        },
        // Laravel Resource biasanya membungkus hasil collection di dalam properti data.
        getProductsFromResponse(response) {
            const responseData = response.data;
            const products = Array.isArray(responseData)
                ? responseData
                : responseData?.data;

            if (!Array.isArray(products)) {
                return [];
            }

            // Abaikan record malformed agar satu respons buruk tidak merusak seluruh tabel.
            return products
                .filter(product => product && typeof product === 'object')
                .map(product => {
                    const price = Number(product.price);

                    return {
                        ...product,
                        price: Number.isFinite(price) ? price : 0
                    };
                });
        },
        getErrorMessage(error, fallbackMessage) {
            const responseData = error.response?.data;
            const validationErrors = responseData?.errors;
            const firstValidationError = validationErrors
                ? Object.values(validationErrors).flat()[0]
                : '';

            return firstValidationError || responseData?.message || fallbackMessage;
        },
        handleRequestError(error, fallbackMessage) {
            if (error.response?.status === 401) {
                // Token tidak valid/kedaluwarsa harus mengembalikan pengguna ke gerbang login.
                this.$emit('unauthorized');
            }

            return this.getErrorMessage(error, fallbackMessage);
        },
        isFormValid() {
            return Boolean(this.formItem.name?.trim())
                && Number.isFinite(Number(this.formItem.price))
                && Number(this.formItem.price) > 0;
        },
        getFormPayload() {
            return {
                name: this.formItem.name.trim(),
                price: Number(this.formItem.price),
                description: this.formItem.description?.trim() || ''
            };
        },
        async fetchProducts(keepLoading = false) {
            this.isLoading = true;
            this.errorMessage = '';
            try {
                const response = await axios.get(
                    `${process.env.VUE_APP_BACKEND}${process.env.VUE_APP_PRODUCTS_ENDPOINT || '/api/products'}`,
                    this.getRequestConfig()
                );
                this.products = this.getProductsFromResponse(response);
            } catch (error) {
                this.errorMessage = this.handleRequestError(error, 'Gagal mengambil data produk.');
            } finally {
                if (!keepLoading) {
                    this.isLoading = false;
                }
            }
        },
        async addItem() {
            if (!this.isLoading && this.isFormValid()) {
                this.isLoading = true;
                this.errorMessage = '';
                this.successMessage = '';
                try {
                    await axios.post(
                        `${process.env.VUE_APP_BACKEND}${process.env.VUE_APP_PRODUCTS_ENDPOINT || '/api/products'}`,
                        this.getFormPayload(),
                        this.getRequestConfig()
                    );
                    await this.fetchProducts(true);
                    this.successMessage = 'Produk berhasil ditambahkan.';
                    this.resetForm();
                } catch (error) {
                    this.errorMessage = this.handleRequestError(error, 'Gagal menambahkan produk.');
                } finally {
                    this.isLoading = false;
                }
            }
        },
        editItem(product) {
            this.isEditing = true;
            this.formItem = { ...product };
            this.currentId = product.id;
        },
        async updateItem() {
            if (!this.isLoading && this.currentId !== null && this.isFormValid()) {
                this.isLoading = true;
                this.errorMessage = '';
                this.successMessage = '';
                try {
                    await axios.put(
                        `${process.env.VUE_APP_BACKEND}${process.env.VUE_APP_PRODUCTS_ENDPOINT || '/api/products'}/${this.currentId}`,
                        this.getFormPayload(),
                        this.getRequestConfig()
                    );
                    await this.fetchProducts(true);
                    this.successMessage = 'Produk berhasil diperbarui.';
                    this.resetForm();
                } catch (error) {
                    this.errorMessage = this.handleRequestError(error, 'Gagal memperbarui produk.');
                } finally {
                    this.isLoading = false;
                }
            }
        },
        cancelEdit() {
            this.resetForm();
        },
        async removeItem(id) {
            if (this.isLoading || !window.confirm('Hapus produk ini?')) {
                return;
            }

            this.isLoading = true;
            this.errorMessage = '';
            this.successMessage = '';
            try {
                await axios.delete(
                    `${process.env.VUE_APP_BACKEND}${process.env.VUE_APP_PRODUCTS_ENDPOINT || '/api/products'}/${id}`,
                    this.getRequestConfig()
                );
                this.products = this.products.filter(product => product.id !== id);
                this.successMessage = 'Produk berhasil dihapus.';
            } catch (error) {
                this.errorMessage = this.handleRequestError(error, 'Gagal menghapus produk.');
            } finally {
                this.isLoading = false;
            }
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
:global(*) {
    box-sizing: border-box;
}

:global(body) {
    margin: 0;
    background: #f5f7f6;
}

.crud-page {
    min-height: 100vh;
    padding: 40px 24px 64px;
    color: #20332e;
}

.page-header,
.dashboard-content {
    width: min(100%, 1120px);
    margin: 0 auto;
}

.page-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 24px;
    padding-bottom: 40px;
}

.eyebrow,
.section-kicker {
    margin: 0 0 8px;
    color: #218c74;
    font-size: 12px;
    font-weight: 800;
    letter-spacing: 0.12em;
    text-transform: uppercase;
}

h1,
h2,
h3,
p {
    margin-top: 0;
}

h1 {
    margin-bottom: 8px;
    font-size: clamp(28px, 4vw, 40px);
    letter-spacing: -0.02em;
}

.page-subtitle {
    margin-bottom: 0;
    color: #687873;
    font-size: 16px;
}

.button,
.action-button {
    border: 0;
    cursor: pointer;
    font: inherit;
    font-weight: 700;
    transition: background-color 180ms ease, border-color 180ms ease, box-shadow 180ms ease, transform 180ms ease;
}

.button {
    min-height: 44px;
    padding: 0 20px;
    border-radius: 6px;
}

.button:hover:not(:disabled),
.action-button:hover:not(:disabled) {
    transform: translateY(-1px);
}

.button:focus-visible,
.action-button:focus-visible,
input:focus-visible {
    outline: 3px solid rgba(33, 140, 116, 0.22);
    outline-offset: 2px;
}

.button:disabled,
.action-button:disabled {
    cursor: not-allowed;
    opacity: 0.55;
}

.button-primary {
    color: #ffffff;
    background: #218c74;
    box-shadow: 0 5px 12px rgba(33, 140, 116, 0.18);
}

.button-primary:hover:not(:disabled) {
    background: #176c5a;
}

.button-secondary,
.button-ghost {
    color: #31443e;
    background: #ffffff;
    border: 1px solid #d7e2de;
}

.button-secondary:hover:not(:disabled),
.button-ghost:hover:not(:disabled) {
    background: #eef5f2;
    border-color: #b7ccc3;
}

.summary-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 16px;
    margin-bottom: 24px;
}

.summary-card,
.editor-section,
.table-section {
    background: #ffffff;
    border: 1px solid #e0e9e5;
    border-radius: 10px;
    box-shadow: 0 10px 30px rgba(32, 51, 46, 0.05);
}

.summary-card {
    min-height: 132px;
    padding: 24px;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.summary-card-accent {
    border-top: 3px solid #218c74;
}

.summary-label,
.summary-note,
.form-state,
.record-count {
    color: #687873;
    font-size: 13px;
}

.summary-card strong {
    margin: 8px 0 4px;
    color: #20332e;
    font-size: 28px;
}

.system-status {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 20px !important;
}

.status-dot,
.loading-indicator {
    width: 8px;
    height: 8px;
    display: inline-block;
    border-radius: 50%;
    background: #42b983;
}

.editor-section,
.table-section {
    padding: 28px;
}

.editor-section {
    margin-bottom: 24px;
}

.section-heading {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 16px;
    margin-bottom: 24px;
}

.section-heading h2 {
    margin-bottom: 0;
    font-size: 21px;
    letter-spacing: -0.01em;
}

.form-state,
.record-count {
    padding: 6px 10px;
    border-radius: 99px;
    background: #eef5f2;
    color: #218c74;
    font-weight: 700;
}

.crud-form {
    display: grid;
    grid-template-columns: 1fr 0.7fr 1.4fr auto;
    align-items: end;
    gap: 16px;
}

.field-group {
    display: flex;
    min-width: 0;
    flex-direction: column;
    gap: 8px;
}

.field-group label {
    color: #31443e;
    font-size: 13px;
    font-weight: 700;
}

.field-group label span {
    color: #91a09b;
    font-weight: 400;
}

input {
    width: 100%;
    min-height: 44px;
    padding: 0 12px;
    border: 1px solid #cbd8d3;
    border-radius: 6px;
    color: #20332e;
    background: #fbfcfc;
    font: inherit;
    transition: border-color 180ms ease, box-shadow 180ms ease;
}

input:focus {
    border-color: #42b983;
    box-shadow: 0 0 0 3px rgba(66, 185, 131, 0.12);
    outline: 0;
}

.input-prefix {
    position: relative;
}

.input-prefix span {
    position: absolute;
    top: 50%;
    left: 12px;
    color: #687873;
    font-size: 13px;
    transform: translateY(-50%);
}

.input-prefix input {
    padding-left: 34px;
}

.form-actions {
    display: flex;
    gap: 8px;
}

.status-message {
    display: flex;
    align-items: center;
    gap: 10px;
    margin: 0 0 24px;
    padding: 12px 16px;
    border: 1px solid;
    border-radius: 6px;
    font-size: 14px;
}

.loading-message {
    border-color: #cbd8d3;
    color: #687873;
    background: #ffffff;
}

.loading-indicator {
    animation: pulse 1s ease-in-out infinite;
}

.error-message {
    border-color: #f1c6c0;
    color: #a93226;
    background: #fff7f6;
}

.success-message {
    border-color: #b9dfce;
    color: #176c5a;
    background: #f2fbf7;
}

.table-heading {
    align-items: center;
    margin-bottom: 20px;
}

.table-wrapper {
    overflow-x: auto;
}

.product-table {
    width: 100%;
    min-width: 720px;
    border-collapse: collapse;
    text-align: left;
}

.product-table th {
    padding: 12px 16px;
    border-bottom: 1px solid #dfe8e4;
    color: #687873;
    font-size: 11px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
}

.product-table td {
    padding: 18px 16px;
    border-bottom: 1px solid #edf2f0;
    color: #53645e;
    font-size: 14px;
}

.product-table tbody tr:last-child td {
    border-bottom: 0;
}

.product-table tbody tr {
    transition: background-color 160ms ease;
}

.product-table tbody tr:hover {
    background: #f8fbfa;
}

.product-id {
    color: #91a09b !important;
    font-variant-numeric: tabular-nums;
}

.product-name {
    color: #20332e !important;
    font-weight: 700;
}

.product-price {
    color: #218c74 !important;
    font-weight: 700;
    white-space: nowrap;
}

.product-description {
    max-width: 300px;
}

.product-actions {
    white-space: nowrap;
    text-align: right;
}

.action-button {
    padding: 7px 10px;
    border-radius: 4px;
    color: #218c74;
    background: transparent;
}

.action-button:hover:not(:disabled) {
    background: #eef5f2;
}

.action-danger {
    color: #b14539;
}

.action-danger:hover:not(:disabled) {
    color: #8f2e25;
    background: #fff2f0;
}

.empty-state {
    padding: 48px 24px;
    border: 1px dashed #cbd8d3;
    border-radius: 6px;
    text-align: center;
}

.empty-icon {
    width: 40px;
    height: 40px;
    display: inline-grid;
    place-items: center;
    margin-bottom: 12px;
    border-radius: 50%;
    color: #218c74;
    background: #eef5f2;
    font-size: 24px;
}

.empty-state h3 {
    margin-bottom: 8px;
    font-size: 17px;
}

.empty-state p {
    margin-bottom: 0;
    color: #687873;
}

.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
}

@keyframes pulse {
    50% {
        opacity: 0.35;
    }
}

@media (max-width: 840px) {
    .crud-form {
        grid-template-columns: repeat(2, minmax(0, 1fr));
    }

    .field-description,
    .form-actions {
        grid-column: 1 / -1;
    }
}

@media (max-width: 560px) {
    .crud-page {
        padding: 24px 16px 40px;
    }

    .page-header {
        align-items: flex-start;
        flex-direction: column;
        padding-bottom: 28px;
    }

    .page-header .button {
        width: 100%;
    }

    .summary-grid {
        grid-template-columns: 1fr;
    }

    .editor-section,
    .table-section {
        padding: 20px 16px;
    }

    .section-heading {
        align-items: flex-start;
        flex-direction: column;
    }

    .crud-form {
        grid-template-columns: 1fr;
    }

    .field-description,
    .form-actions {
        grid-column: auto;
    }

    .form-actions,
    .form-actions .button {
        width: 100%;
    }
}
</style>