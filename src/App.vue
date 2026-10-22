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
    }
}





</script>