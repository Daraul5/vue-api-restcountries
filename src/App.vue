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
        
        // Detectamos si estamos en producción (GitHub Pages) o en local
        const isProduction = window.location.hostname.includes('github.io');
        
        let url = `https://api.restcountries.com/countries/v5?q=${query}`;
        
        // Si estamos en GitHub Pages, envolvemos la petición en el proxy CORS para evitar el bloqueo del navegador
        if (isProduction) {
            const targetUrl = encodeURIComponent(`${url}&api-key=rc_live_5ca1639020f94dbe9464c3adf8f77759`);
            url = `https://corsproxy.io/?url=${targetUrl}`;
        }
        
        const options = isProduction ? {} : {
            method: 'GET',
            headers: {
                'Authorization': 'Bearer rc_live_5ca1639020f94dbe9464c3adf8f77759'
            }
        };
        
        const response = await fetch(url, options);
        
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

<style scoped>
.container { max-width: 1200px; margin: 0 auto; padding: 20px; font-family: Arial, sans-serif; }
h1 { text-align: center; color: #333; }
.search-container { display: flex; justify-content: center; margin-bottom: 20px; }
.search-box { width: 100%; max-width: 400px; padding: 10px; font-size: 16px; border: 1px solid #ccc; border-radius: 5px; }
.status-message { text-align: center; font-size: 18px; margin: 40px 0; }
.error { color: red; } .loading { color: #0066cc; } .empty { color: #666; }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; }
.card { border: 1px solid #ddd; border-radius: 8px; overflow: hidden; box-shadow: 0 4px 6px rgba(0,0,0,0.1); background: white; }
.flag { width: 100%; height: 150px; object-fit: cover; border-bottom: 1px solid #ddd; }
.card-info { padding: 15px; }
.card-info h2 { margin: 0 0 10px 0; font-size: 20px; }
.card-info p { margin: 5px 0; color: #555; }
</style>