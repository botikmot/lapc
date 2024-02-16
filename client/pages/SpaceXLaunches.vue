<template>
    <v-container>
        <h1 class="mb-1">SpaceX Launches</h1>
        <div v-if="loading"><h3>Loading..</h3></div>
        <div v-else>
            <div v-for="launch in visibleLaunches" :key="launch.id">
                <h2>Mission: {{ launch.mission_name }}</h2>
                <p><strong>Launch Date:</strong> {{ formatDate(launch.launch_date_utc) }}</p>
                <p><strong>Launch Site:</strong> {{ launch.launch_site ? launch.launch_site.site_name : '' }}</p>
                <p><strong>Rocket:</strong> {{ launch.rocket.rocket_name }}</p>
                <p v-if="launch.details"><strong>Details:</strong> {{ launch.details }}</p>
                <hr>
            </div>
            <div class="d-flex justify-space-between mt-1">
                <v-btn @click="previousPage" :disabled="currentPage === 1">Previous</v-btn>
                <div>Page: {{ currentPage }} of {{ numberOfPages }}</div>
                <v-btn @click="nextPage" :disabled="currentPage * itemsPerPage >= launches.length">Next</v-btn>
            </div>
        </div>
    </v-container>
</template>

<script lang="ts" setup>

    let loading = ref(true)
    let launches = ref([]);
    let currentPage = ref(1);
    const itemsPerPage = 5; // Number of launches per page
  
    const query = gql`
        query {
                launches {
                id
                mission_name
                launch_date_utc
                launch_site {
                    site_name
                }
                rocket {
                    rocket_name
                }
                details
                }
            }
        `;
    const formatDate = (dateString) => {
        const date = new Date(dateString);
        return date.toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
    }

    async function fetchData() {
        const { data } = await useAsyncQuery(query);
        loading.value = false;
        launches.value = data.value?.launches;
    }

    const visibleLaunches = computed(() => {
        const startIndex = (currentPage.value - 1) * itemsPerPage;
        const endIndex = startIndex + itemsPerPage;
        return launches.value.slice(startIndex, endIndex);
    });

    const numberOfPages = computed(() => Math.ceil(launches.value.length / itemsPerPage));

    const nextPage = () => {
        currentPage.value++;
    }

    const previousPage = () => {
        currentPage.value--;
    }

    fetchData(); // Fetch data when component is loaded
</script>
  
  
  