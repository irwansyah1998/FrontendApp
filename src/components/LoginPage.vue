<template>
    <main class="login-page">
        <!-- Layout login diperbarui menjadi panel dua sisi tanpa menyentuh proses autentikasi. -->
        <section class="login-shell" aria-labelledby="login-title">
            <div class="login-intro">
                <span class="brand-mark">B</span>
                <span class="login-eyebrow">BackendApp workspace</span>
                <h1>Kelola katalog dengan lebih terarah.</h1>
                <p>Tempat kerja sederhana untuk menjaga produk tetap rapi, terbarui, dan siap digunakan.</p>
                <div class="intro-line"></div>
                <span class="intro-meta">Secure product management</span>
            </div>

            <div class="login-card">
                <div class="login-heading">
                    <span class="login-eyebrow">Selamat datang kembali</span>
                    <h2 id="login-title">Masuk ke akun Anda</h2>
                    <p>Gunakan kredensial BackendApp untuk melanjutkan.</p>
                </div>

                <form @submit.prevent="login" class="login-form">
                    <div class="field-group">
                        <label for="email">Email</label>
                        <input
                            id="email"
                            v-model.trim="credentials.email"
                            type="email"
                            autocomplete="email"
                            placeholder="nama@perusahaan.com"
                            required
                            :disabled="isLoading"
                        />
                    </div>

                    <div class="field-group">
                        <label for="password">Password</label>
                        <input
                            id="password"
                            v-model="credentials.password"
                            type="password"
                            autocomplete="current-password"
                            placeholder="Masukkan password"
                            required
                            :disabled="isLoading"
                        />
                    </div>

                    <p v-if="errorMessage" class="status-message error-message" role="alert">
                        {{ errorMessage }}
                    </p>

                    <button type="submit" class="login-button" :disabled="isLoading">
                        {{ isLoading ? 'Memproses...' : 'Masuk ke workspace' }}
                    </button>
                </form>
            </div>
        </section>
    </main>
</template>

<script>
import axios from 'axios';

export default {
    name: 'LoginPage',
    data() {
        return {
            credentials: { email: '', password: '' },
            isLoading: false,
            errorMessage: ''
        };
    },
    methods: {
        // Mendukung format token umum dari endpoint login Laravel.
        getTokenFromResponse(response) {
            const responseData = response.data;
            const data = responseData?.data || responseData;

            return data?.token || data?.access_token || '';
        },
        getErrorMessage(error) {
            const responseData = error.response?.data;
            const validationErrors = responseData?.errors;
            const firstValidationError = validationErrors
                ? Object.values(validationErrors).flat()[0]
                : '';

            return firstValidationError || responseData?.message || 'Login gagal. Periksa email dan password Anda.';
        },
        async login() {
            this.isLoading = true;
            this.errorMessage = '';

            try {
                const response = await axios.post(
                    `${process.env.VUE_APP_BACKEND}${process.env.VUE_APP_LOGIN_ENDPOINT || '/api/login'}`,
                    this.credentials,
                    { headers: { Accept: 'application/json' } }
                );
                const token = this.getTokenFromResponse(response);

                if (!token) {
                    throw new Error('Login gagal. Respons server tidak valid.');
                }

                // Token disimpan hanya di browser agar request produk dapat terautentikasi.
                localStorage.setItem('backendapp_api_token', token);
                this.$emit('authenticated', token);
            } catch (error) {
                this.errorMessage = error.response
                    ? this.getErrorMessage(error)
                    : 'Login gagal. Periksa koneksi dan coba lagi.';
            } finally {
                this.isLoading = false;
            }
        }
    }
};
</script>

<style scoped>
.login-page {
    min-height: 100vh;
    display: grid;
    place-items: center;
    padding: 32px 24px;
    box-sizing: border-box;
    background: #f5f7f6;
}

.login-shell {
    display: grid;
    grid-template-columns: minmax(260px, 0.9fr) minmax(340px, 1.1fr);
    width: min(100%, 880px);
    min-height: 520px;
    overflow: hidden;
    border: 1px solid #dfe8e4;
    border-radius: 12px;
    background: #ffffff;
    box-shadow: 0 20px 50px rgba(32, 51, 46, 0.1);
}

.login-intro {
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 48px;
    color: #ffffff;
    background: #203f36;
}

.brand-mark {
    width: 42px;
    height: 42px;
    display: grid;
    place-items: center;
    margin-bottom: 48px;
    border: 1px solid rgba(255, 255, 255, 0.28);
    border-radius: 8px;
    color: #ffffff;
    font-size: 20px;
    font-weight: 800;
}

.login-intro .login-eyebrow {
    color: #8de0bc;
}

.login-intro h1 {
    margin: 14px 0 16px;
    font-size: clamp(28px, 4vw, 38px);
    line-height: 1.1;
    letter-spacing: -0.025em;
}

.login-intro p {
    max-width: 330px;
    margin-bottom: 32px;
    color: #c7dcd4;
    line-height: 1.6;
}

.intro-line {
    width: 48px;
    height: 3px;
    margin-bottom: 16px;
    background: #42b983;
}

.intro-meta {
    color: #9dbbb0;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
}

.login-card {
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 48px;
    text-align: left;
}

.login-eyebrow {
    color: #218c74;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 0.12em;
    text-transform: uppercase;
}

h2 {
    margin: 12px 0 8px;
    color: #20332e;
    font-size: 30px;
    letter-spacing: -0.02em;
}

.login-heading p {
    margin: 0 0 32px;
    color: #687873;
    line-height: 1.5;
}

.login-form {
    display: grid;
    gap: 20px;
}

.field-group {
    display: grid;
    gap: 8px;
}

label {
    color: #31443e;
    font-size: 14px;
    font-weight: 600;
}

input {
    width: 100%;
    min-height: 46px;
    padding: 0 12px;
    box-sizing: border-box;
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

.login-button {
    min-height: 46px;
    margin-top: 4px;
    padding: 0 16px;
    color: #ffffff;
    background: #218c74;
    border: 0;
    border-radius: 6px;
    cursor: pointer;
    font: inherit;
    font-weight: 700;
    transition: background-color 180ms ease, box-shadow 180ms ease, transform 180ms ease;
}

.login-button:hover:not(:disabled) {
    background: #176c5a;
    box-shadow: 0 6px 16px rgba(33, 140, 116, 0.2);
    transform: translateY(-1px);
}

.login-button:disabled {
    cursor: wait;
    opacity: 0.65;
}

.status-message {
    margin: 8px 0 0;
    font-size: 14px;
}

.error-message {
    margin: 0;
    padding: 12px;
    border: 1px solid #f1c6c0;
    border-radius: 6px;
    color: #c0392b;
    background: #fff7f6;
}

@media (max-width: 700px) {
    .login-page {
        padding: 16px;
    }

    .login-shell {
        grid-template-columns: 1fr;
        min-height: auto;
    }

    .login-intro {
        padding: 32px;
    }

    .brand-mark {
        margin-bottom: 32px;
    }

    .login-intro h1 {
        font-size: 30px;
    }

    .login-card {
        padding: 32px;
    }
}
</style>
