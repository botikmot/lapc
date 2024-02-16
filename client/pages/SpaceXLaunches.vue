<template>
    <v-container>
        <v-row>
            <v-col>
                <h1 class="mb-1">SpaceX Launches</h1>
            </v-col>
            <v-col cols="2" class="d-flex">
                <v-select class="w-25" v-model="selectedYear" :items="allYears" label="Filter by Year" @change="filterLaunchesByYear" />
            </v-col>
            <v-col cols="4" class="d-flex">
                <!-- Dropdown menu for selecting sorting type -->
                <v-select v-model="sortType" :items="sortTypes" item-title="name" item-value="value" label="Sort By" @change="sortLaunches" />
                <!-- Buttons for selecting sorting order -->
                <v-btn-toggle v-model="sortOrder">
                    <v-btn value="asc">Asc</v-btn>
                    <v-btn value="desc">Desc</v-btn>
                </v-btn-toggle>
            </v-col>
        </v-row>
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
    let tempLaunches = ref([]);
    let currentPage = ref(1);
    const itemsPerPage = 5; // Number of launches per page
    let selectedYear = ref('');
    let sortType = ref('mission_name'); // Default sorting type is by mission name
    let sortOrder = ref('asc'); // Default sorting order is ascending
    const sortTypes = ref([
        {
            name: 'Mission name',
            value: 'mission_name',
        },
        {
            name: 'Date',
            value: 'launch_date_utc',
        }
    ])
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
        tempLaunches.value = data.value?.launches;
    }

    const filteredLaunches = computed(() => {
        if (selectedYear.value === '') {
            return launches.value;
        } else {
            return launches.value.filter(launch => {
                const launchYear = new Date(launch.launch_date_utc).getFullYear().toString();
                return launchYear === selectedYear.value;
            });
        }
    });

    const visibleLaunches = computed(() => {
        const startIndex = (currentPage.value - 1) * itemsPerPage;
        const endIndex = startIndex + itemsPerPage;
        return filteredLaunches.value.slice(startIndex, endIndex);
    });

    const numberOfPages = computed(() => Math.ceil(launches.value.length / itemsPerPage));

    const nextPage = () => {
        currentPage.value++;
    }

    const previousPage = () => {
        currentPage.value--;
    }

    const filterLaunchesByYear = () => {
        if (selectedYear.value === '') {
            fetchData();
        } else {
            launches.value = tempLaunches.value.filter(launch => {
            const launchYear = new Date(launch.launch_date_utc).getFullYear().toString();
            return launchYear === selectedYear.value;
            });
            currentPage.value = 1;
        }
    }

    // Function to sort launches
    const sortLaunches = () => {
        console.log('sort type', sortType.value)
        if (sortType.value === 'mission_name') {
            sortLaunchesByMissionName();
        } else if (sortType.value === 'launch_date_utc') {
            sortLaunchesByDate();
        }
    }

    watch(selectedYear, filterLaunchesByYear);
    // Watcher for changes in sorting order
    watch(sortOrder, sortLaunches);
    // Fetch available years
    const allYears = computed(() => {
    const years = new Set();
        tempLaunches.value.forEach(launch => {
            const year = new Date(launch.launch_date_utc).getFullYear().toString();
            years.add(year);
        });
        return Array.from(years);
    });

    // Function to sort launches by mission name
    const sortLaunchesByMissionName = () => {
        console.log('Sorting launches by mission name');
        const sortedLaunches = [...launches.value]; // Create a copy of the original launches array
        sortedLaunches.sort((a, b) => {
            if (sortOrder.value === 'asc') {
                return a.mission_name.localeCompare(b.mission_name);
            } else {
                return b.mission_name.localeCompare(a.mission_name);
            }
        });
        launches.value = sortedLaunches; // Assign the sorted copy back to launches.value
    }

    // Function to sort launches by launch date
    const sortLaunchesByDate = () => {
        console.log('Sorting launches by date');
        const sortedLaunches = [...launches.value]; // Create a copy of the original launches array
        sortedLaunches.sort((a, b) => {
            const dateA = new Date(a.launch_date_utc);
            const dateB = new Date(b.launch_date_utc);
            if (sortOrder.value === 'asc') {
                return dateA - dateB;
            } else {
                return dateB - dateA;
            }
        });
        launches.value = sortedLaunches; // Assign the sorted copy back to launches.value
    }


    fetchData(); // Fetch data when component is loaded
</script>
  
  
  