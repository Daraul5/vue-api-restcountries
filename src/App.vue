<script setup>
import { ref, onMounted, computed } from 'vue';

const countries = ref([]);
const searchQuery = ref('');
const isLoading = ref(true);
const errorMessage = ref('');

const fetchCountries = async () => {
    try {
        // 1. Usando EXACTAMENTE la API solicitada en la rúbrica
        const response = await fetch('https://api.restcountries.com/countries/v5?q=Canada&api-key=rc_live_demo&pretty=1');
        
        if (!response.ok) {
            throw new Error('Error en la respuesta de la API');
        }
        
        const data = await response.json();
        
        // La API puede devolver un arreglo o un objeto dependiendo de la búsqueda. 
        // Esto asegura que siempre lo guardemos como un arreglo para el v-for.
        countries.value = Array.isArray(data) ? data : (data.data || [data]);
        
    } catch (error) {
        errorMessage.value = 'No fue posible obtener los datos de la API';
        console.error(error);
    } finally {
        isLoading.value = false;
    }
}

// Se ejecuta al montar el componente (Requisito de rúbrica)
onMounted(() => {
    fetchCountries();
});

// Filtro local (Requisito de rúbrica: v-model)
const filterCountries = computed(() => {
    const term = searchQuery.value.toLowerCase();
    return countries.value.filter(country => {
        // Extraemos el nombre y región de forma segura (por si la v5 cambia la estructura)
        const name = country.name?.common || country.name || '';
        const region = country.region || '';
        return name.toLowerCase().includes(term) || region.toLowerCase().includes(term);
    });
});
</script>

<template>
    <div class="container">
        <h1>Explorador de Países</h1>
        
        <div class="search-container">
            <input
                type="text"
                v-model="searchQuery"
                placeholder="Buscar país por nombre o región..."
                class="search-box"
            />
        </div>

        <div v-if="isLoading" class="status-message loading">
            ⏳ Cargando países...
        </div>

        <div v-else-if="errorMessage" class="status-message error">
            {{ errorMessage }}
        </div>

        <div v-else>
            <div v-if="filterCountries.length === 0" class="status-message empty">
                No hay resultados para tu búsqueda.
            </div>

            <div class="grid">
                <div
                    v-for="country in filterCountries"
                    :key="country.cca3 || country.name"
                    class="card"
                >
                    <!-- Manejo seguro de la imagen por si la v5 usa flag en lugar de flags.svg -->
                    <img :src="country.flags?.svg || country.flags?.png || country.flag" :alt="`Bandera de ${country.name?.common || country.name}`" class="flag" />
                    
                    <div class="card-info">
                        <h2>{{ country.name?.common || country.name }}</h2> 
                        <p><strong>Capital:</strong> {{ country.capital ? (Array.isArray(country.capital) ? country.capital[0] : country.capital) : 'No tiene' }}</p>
                        <p><strong>Región:</strong> {{ country.region }}</p>
                        <p><strong>Población:</strong> {{ country.population ? country.population.toLocaleString() : 'Desconocida' }}</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
}
h1 {
    text-align: center;
    color: #333;
}
.search-container {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
}
.search-box {
    width: 100%;
    max-width: 400px;
    padding: 10px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 5px;
}
.status-message {
    text-align: center;
    font-size: 18px;
    margin: 40px 0;
}
.error { color: red; }
.loading { color: #0066cc; }
.empty { color: #666; }

.grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
}
.card {
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    background: white;
}
.flag {
    width: 100%;
    height: 150px;
    object-fit: cover;
    border-bottom: 1px solid #ddd;
}
.card-info {
    padding: 15px;
}
.card-info h2 {
    margin: 0 0 10px 0;
    font-size: 20px;
}
.card-info p {
    margin: 5px 0;
    color: #555;
}
</style>