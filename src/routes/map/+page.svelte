<svelte:head>
  <link rel="stylesheet" href="/css/map.css" />
</svelte:head>

<script lang="ts">
import { onMount, onDestroy } from 'svelte';
import { browser } from '$app/environment';
import { transliterate } from 'transliteration';
import { getUserLocation } from '$lib/usersLocation';
import type { UserLocation } from '$lib/usersLocation';

let mapContainer: HTMLDivElement | null = null;
let map: any;
let markers: any[] = [];
let selectedCategory = "all";
let searchQuery = "";

let userLocation: UserLocation | null = null;
let isNearMeActive = false;


// chita category od URL 
$: if (browser) {
    const urlParams = new URLSearchParams(window.location.search);
    selectedCategory = urlParams.get('category') || "all";
}

const fetchOverpassData = async () => {
    const query = `
    [out:json];
    area[name="Скопје"]->.a;
    (
      node["amenity"="restaurant"](area.a);
      node["amenity"="cafe"](area.a);
      node["amenity"="bar"](area.a);
      node["amenity"="pub"](area.a);
    );
    out body;
    `;
    const response = await fetch(`https://overpass-api.de/api/interpreter?data=${encodeURIComponent(query)}`);
    const data = await response.json();
    return data.elements;
};

const updateMarkers = async (locations: any[]) => {
    const leaflet = await import('leaflet');
    markers.forEach(marker => map.removeLayer(marker));
    markers = [];

    const icons: { [key: string]: any } = {
        restaurant: leaflet.icon({ iconUrl: '/icons/restaurant.png', iconSize: [32, 32] }),
        cafe: leaflet.icon({ iconUrl: '/icons/cafe.png', iconSize: [36, 36] }),
        bar: leaflet.icon({ iconUrl: '/icons/bar.png', iconSize: [32, 32] }),
        pub: leaflet.icon({ iconUrl: '/icons/pub.png', iconSize: [32, 32] }),
    };

    const filteredLocations = locations.filter(location => {
        const hasName = location.tags.name && location.tags.name.trim().length > 0;
        if (!hasName) return false;

        const locationNameTranslit = transliterate(location.tags.name.toLowerCase());
        const searchQueryTranslit = transliterate(searchQuery.toLowerCase());

        const matchesCategory = selectedCategory === "all" || location.tags.amenity === selectedCategory;
        const matchesSearch = locationNameTranslit.includes(searchQueryTranslit);

        if (isNearMeActive && userLocation) {
        const distance = getDistanceFromLatLonInKm(
            userLocation.lat,
            userLocation.lng,
            location.lat,
            location.lon
        );
        if (distance > 1.5) return false;
    }

        return matchesCategory && matchesSearch;
    });
    
    function getDistanceFromLatLonInKm(lat1: number, lon1: number, lat2: number, lon2: number): number {
    const R = 6371; // Radius of the earth in km
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a =
        0.5 - Math.cos(dLat) / 2 +
        Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
        (1 - Math.cos(dLon)) / 2;
    return R * 2 * Math.asin(Math.sqrt(a));
}


    filteredLocations.forEach(location => {
        if (location.lat && location.lon) {
            const category = location.tags.amenity || "restaurant";
            const icon = icons[category] || icons['restaurant'];

            const marker = leaflet.marker([location.lat, location.lon], { icon });
            const googleMapsUrl = `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(location.tags.name)}&near=${location.lat},${location.lon}&radius=1000`;

            marker.bindPopup(`
                <strong>${location.tags.name}</strong><br>
                <a href="${googleMapsUrl}" target="_blank">View on Google Maps</a>
            `).addTo(map);

            markers.push(marker);
        }
    });

    console.log("Markers updated:", markers.length);
};

const handleFilterChange = async (event: Event) => {
    selectedCategory = (event.target as HTMLSelectElement).value;
    const url = new URL(window.location.href);
    url.searchParams.set('category', selectedCategory);
    window.history.pushState({}, "", url);
    const locations = await fetchOverpassData();
    await updateMarkers(locations);
};

const handleSearch = async () => {
    const locations = await fetchOverpassData();
    await updateMarkers(locations);
};

onMount(async () => {
    if (browser && mapContainer) {
        const leaflet = await import('leaflet');
        await import('leaflet/dist/leaflet.css');

        map = leaflet.map(mapContainer).setView([41.9981, 21.4254], 13);

        leaflet.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors'
        }).addTo(map);

        const locations = await fetchOverpassData();
        await updateMarkers(locations);
    }
});
$: if (browser && map && selectedCategory) {
    fetchOverpassData().then(updateMarkers);
}

const toggleNearMe = async () => {
    isNearMeActive = !isNearMeActive;

    if (isNearMeActive) {
        userLocation = await getUserLocation();
        if (!userLocation) {
            alert("Could not get your location.");
            isNearMeActive = false;
            return;
        }
    }

    const locations = await fetchOverpassData();
    await updateMarkers(locations);
};


onDestroy(() => {
    if (map) map.remove();
});

</script>

<main>
    <div class="filter-container">
        <label for="category-filter">Filter by Type:</label>
        <select id="category-filter" on:change={handleFilterChange} bind:value={selectedCategory}>
            <option value="all">All</option>
            <option value="restaurant">Restaurants</option>
            <option value="cafe">Cafes</option>
            <option value="bar">Bars</option>
            <option value="pub">Pubs</option>
        </select>
        <button on:click={toggleNearMe}>Near Me</button>
    </div>
    
    <!-- Search bar -->
    <div class="search-container">
        <input type="text" bind:value={searchQuery} placeholder="Search by name..." />
        <button on:click={handleSearch}>Search</button>
    </div>
    <!-- Map -->
    <div class="map-container" bind:this={mapContainer}></div>
    
</main>

<style>
     @import 'leaflet/dist/leaflet.css';
     /* @import '/css/map.css'; */
</style>
