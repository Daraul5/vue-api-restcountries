<script setup>
import { ref, onMounted, watch } from 'vue';

const countries = ref([]);
const searchQuery = ref('');
const isLoading = ref(true);
const errorMessage = ref('');

const fetchCountries = async (termino) => {
    isLoading.value = true;
    errorMessage.value = '';
    
    try {
        const query = termino ? termino : 'Canada';
        
        // Enviamos la clave directamente en la URL para evitar el bloqueo de Headers en GitHub Pages
        const url = `https://api.restcountries.com/countries/v5?q=${query}&api-key=rc_live_5ca1639020f94dbe9464c3adf8f77759`;
        
        const response = await fetch(url);
        
        if (!response.ok) {
            if (response.status === 404 || response.status === 400) {
                countries.value = [];
                return;
            }
            throw new Error(`Error HTTP: ${response.status}`);
        }
        
        const json = await response.json();
        countries.value = json.data?.objects || [];
        
    } catch (error) {
        errorMessage.value = 'No fue posible obtener los datos de la API';
        console.error("Error en la petición:", error);
    } finally {
        isLoading.value = false;
    }
}

// Carga inicial al montar el componente (Requisito de rúbrica)
onMounted(() => {
    fetchCountries('');
});

// Búsqueda dinámica con temporizador (debounce)
let temporizador;
watch(searchQuery, (nuevoValor) => {
    clearTimeout(temporizador);
    temporizador = setTimeout(() => {
        fetchCountries(nuevoValor.trim());
    }, 400);
});

const clearSearch = () => {
    searchQuery.value = '';
};
</script>

<template>
    <div class="container">
        <header class="header">
            <span class="header-badge">Directorio Global</span>
            <h1>Explorador de Países</h1>
            <p class="subtitle">Consulta información geográfica y demográfica en tiempo real</p>
        </header>
        
        <div class="search-section">
            <div class="search-wrapper">
                <svg class="search-icon" aria-hidden="true" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="11" cy="11" r="8"></circle>
                    <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
                </svg>
                <input
                    type="text"
                    v-model="searchQuery"
                    placeholder="Busca un país o región..."
                    class="search-box"
                    aria-label="Buscar país o región"
                />
                <button
                    v-if="searchQuery"
                    @click="clearSearch"
                    type="button"
                    class="clear-btn"
                    title="Limpiar búsqueda"
                    aria-label="Limpiar búsqueda"
                >
                    &times;
                </button>
            </div>
            
            <div v-if="!isLoading && !errorMessage && countries.length > 0" class="search-meta">
                Mostrando <strong>{{ countries.length }}</strong> {{ countries.length === 1 ? 'país' : 'países' }}
            </div>
        </div>

        <!-- Estado de carga -->
        <div v-if="isLoading" class="status-message loading">
            <div class="spinner" aria-hidden="true"></div>
            <p>Buscando países...</p>
        </div>

        <!-- Estado de error -->
        <div v-else-if="errorMessage" class="status-message error-box">
            <p>{{ errorMessage }}</p>
            <button @click="fetchCountries(searchQuery.trim())" type="button" class="retry-btn">
                Reintentar
            </button>
        </div>

        <!-- Resultados / Estado vacío -->
        <div v-else>
            <div v-if="countries.length === 0" class="status-message empty-box">
                <h3>Sin resultados</h3>
                <p>No se encontraron países que coincidan con "{{ searchQuery }}".</p>
            </div>

            <div v-else class="grid">
                <article
                    v-for="country in countries"
                    :key="country.codes?.alpha_3 || country.uuid || country.names?.common"
                    class="card"
                >
                    <div class="flag-wrapper">
                        <img 
                            :src="country.flag?.url_svg" 
                            :alt="`Bandera de ${country.names?.common}`" 
                            class="flag" 
                            loading="lazy"
                        />
                        <span v-if="country.region" class="region-chip">
                            {{ country.region }}
                        </span>
                    </div>
                    
                    <div class="card-info">
                        <h2>{{ country.names?.common }}</h2> 
                        <ul class="country-details">
                            <li>
                                <span class="detail-label">Capital</span>
                                <span class="detail-value">{{ country.capitals?.[0]?.name || 'No tiene' }}</span>
                            </li>
                            <li>
                                <span class="detail-label">Región</span>
                                <span class="detail-value">{{ country.region || 'Desconocida' }}</span>
                            </li>
                            <li>
                                <span class="detail-label">Población</span>
                                <span class="detail-value">{{ country.population ? country.population.toLocaleString() : 'Desconocida' }}</span>
                            </li>
                        </ul>
                    </div>
                </article>
            </div>
        </div>

        <footer class="footer">
            <p>Proyecto 2 Web &bull; Conexión a REST Countries API</p>
        </footer>
    </div>
</template>