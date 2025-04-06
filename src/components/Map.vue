<template>
    <div class="map-container">
        <div id="map"></div>
        <div v-if="error" class="error-message">
            {{ error }}
        </div>
    </div>
</template>

<script setup lang="ts">
import leaflet from "leaflet"
import { onMounted, ref, watchEffect } from "vue"
import { useGeolocation } from "@vueuse/core"

// Import Leaflet CSS
import "leaflet/dist/leaflet.css"

let map: leaflet.Map | null = null
let userGeoMarker: leaflet.Marker | null = null
const error = ref<string | null>(null)
const { coords, isSupported, error: geoError } = useGeolocation()

// Define reasonable bounds for the map
const worldBounds = leaflet.latLngBounds(
    leaflet.latLng(-85, -180), // Southwest corner
    leaflet.latLng(85, 180)    // Northeast corner
)

// Create a larger icon
const largeIcon = leaflet.icon({
    iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
    iconSize: [76, 120],
    iconAnchor: [12, 41],
    popupAnchor: [1, -34],
    shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
    shadowSize: [41, 41]
})

function isValidCoordinate(lat: number, lng: number): boolean {
    return !isNaN(lat) && !isNaN(lng) && 
           lat >= -90 && lat <= 90 && 
           lng >= -180 && lng <= 180
}

onMounted(() => {
    console.log("Map component mounted")
    
    try {
        // Initialize map with default coordinates
        const mapElement = document.getElementById("map")
        console.log("Map element:", mapElement)
        
        if (!mapElement) {
            throw new Error("Map element not found")
        }

        map = leaflet.map("map", {
            center: [58.3779, 26.7290], // Tartu coordinates
            zoom: 13,
            maxBounds: worldBounds,
            maxBoundsViscosity: 1.0,
            minZoom: 2,
            maxZoom: 19
        })

        console.log("Map initialized:", map)

        leaflet.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
            maxZoom: 19,
            attribution:
                '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
        }).addTo(map)

        // If geolocation is available, update the map view
        if (isSupported.value && coords.value) {
            const { latitude, longitude } = coords.value
            if (isValidCoordinate(latitude, longitude)) {
                updateUserLocation(latitude, longitude)
            } else {
                console.warn("Invalid initial coordinates:", { latitude, longitude })
            }
        } else {
            console.log("Geolocation not available")
        }

        if (geoError.value) {
            error.value = "Geograafilise asukoha määramine ebaõnnestus"
            console.error("Geolocation error:", geoError.value)
        }
    } catch (e) {
        error.value = "Kaardi initsialiseerimine ebaõnnestus"
        console.error("Map initialization error:", e)
    }
})

function updateUserLocation(latitude: number, longitude: number) {
    if (!map) return

    if (!isValidCoordinate(latitude, longitude)) {
        console.warn("Invalid coordinates received:", { latitude, longitude })
        return
    }

    // Remove existing marker if it exists
    if (userGeoMarker) {
        userGeoMarker.remove()
    }
    
    // Add new marker with larger icon
    userGeoMarker = leaflet.marker([latitude, longitude], { icon: largeIcon })
        .addTo(map)
        .bindPopup("Teie asukoht")
        .openPopup()
}

watchEffect(() => {
    if (coords.value) {
        const { latitude, longitude } = coords.value
        if (isValidCoordinate(latitude, longitude)) {
            updateUserLocation(latitude, longitude)
        } else {
            console.warn("Invalid coordinates in watchEffect:", { latitude, longitude })
        }
    }
})
</script>

<style>
.map-container {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
}

#map {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
}

.error-message {
    position: absolute;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    background-color: rgba(255, 0, 0, 0.8);
    color: white;
    padding: 10px 20px;
    border-radius: 5px;
    z-index: 1000;
}
</style>