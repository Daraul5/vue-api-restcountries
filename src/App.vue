<script setup>
import {ref, onMounted, computed} from 'vue';

const countries = ref([]);
const searchQuery = ref('');
const isLoading = ref(true);
const errorMessage = ref('');

const fetchCountries = async () => {
    try{
        const response = await fetch('https://restcountries.com/v3.1/all')
        if (!response.ok){
            throw new Error('ERROR DE LA RESPUESTA DEL SERVIDOR');
        }
        const data = await response.json();
        countries.value = data;
    }catch (error){
        errorMessage.value = 'No fue posible obtener los datos de la api';
        console.error(error);
    }finally{
        isLoading.value=false
    }
}

onMounted(() => {
    fetchCountries()
});

const filterCountries = computed(() => {
    const term = searchQuery.value.toLowerCase();
    return countries.value.filter(country => 
    country.name.common.toLowerCase().includes(term) 
    || country.region.toLowerCase().includes(term))
})
</script>

<template>
    <div class="container">
        <h1>Explorador de Paises</h1>
        <div class="search-container">
            <input
                type="text"
                v-model="searchQuery"
                placeholder="Buscar pais por nombre o region"
                class="search-box"
            />
        </div>
        <div v-if="isLoading" class="status-message error">
            {{ errorMessage }}
        </div>
        <div v-else>
            <div v-if="filterCountries.length === 0" class="status-message empty">
                No hay resultado para tu busqueda
            </div>
        </div>
        <div class="grid">
            <div
                v-for="country in filterCountries"
                :key="country.cca3"
                class="card"
            >
                <img :src="country.flags.svg" :alt="`Bandera de ${country.name.common}`" class="flag" />
                <div class="card-info">
                    <p><strong>Capital:</strong> {{ country.capital ? country.capital[0] : 'No tiene' }}</p>
                    <p><strong>Región:</strong> {{ country.region }}</p>
                    <p><strong>Población:</strong> {{ country.population.toLocaleString() }}</p>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* Estilos básicos para la interfaz */
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