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
</script>

<template>
    <div class="container">
        <h1>Explorador de Países</h1>
        
        <div class="search-container">
            <input
                type="text"
                v-model="searchQuery"
                placeholder="Busca un país o región..."
                class="search-box"
            />
        </div>

        <div v-if="isLoading" class="status-message loading">
            ⏳ Buscando...
        </div>

        <div v-else-if="errorMessage" class="status-message error">
            {{ errorMessage }}
        </div>

        <div v-else>
            <div v-if="countries.length === 0" class="status-message empty">
                No hay resultados para tu búsqueda.
            </div>

            <div class="grid">
                <div
                    v-for="country in countries"
                    :key="country.codes?.alpha_3"
                    class="card"
                >
                    <img 
                        :src="country.flag?.url_svg" 
                        :alt="`Bandera de ${country.names?.common}`" 
                        class="flag" 
                    />
                    
                    <div class="card-info">
                        <h2>{{ country.names?.common }}</h2> 
                        <p><strong>Capital:</strong> {{ country.capitals?.[0]?.name || 'No tiene' }}</p>
                        <p><strong>Región:</strong> {{ country.region }}</p>
                        <p><strong>Población:</strong> {{ country.population ? country.population.toLocaleString() : 'Desconocida' }}</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
@import './style.css';
</style>