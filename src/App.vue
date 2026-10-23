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
            
        </div>    
    </div>
</template>
