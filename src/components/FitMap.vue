<template>
    <div style="height:600px; width:800px">
        <l-map ref="map" :zoom="zoom" :center="[0, 0]">
            <l-tile-layer
                url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
                layer-type="base"
                name="OpenStreetMap"
            ></l-tile-layer>

            <l-polyline
                ref="polyline"
                :lat-lngs="coords"
                color="green"
            />
        </l-map>
    </div>
    <input type="file" ref="inputFile" id="inputFile" @change="handleFileChange"/>
</template>

<script lang="ts">
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LPolyline } from "@vue-leaflet/vue-leaflet";
import { Decoder, Stream, Profile } from '@garmin/fitsdk'



export default {
    components: {
        LMap,
        LTileLayer,
        LPolyline
    },

    data() {
        return {
            zoom: 0,
            coords: []
        };
    },
    methods: {
        handleFileChange(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();

                reader.onload = () => {
                    const streamfromFileSync = Stream.fromArrayBuffer(reader.result)
                    const decoder = new Decoder(streamfromFileSync);

                    const coords = new Map<Object, Array<number>>();
                    const onMesg = (messageNumber, message) => {
                        if (Profile.types.mesgNum[messageNumber] === "record") {
                            coords.set(message.timestamp, [message.positionLat / 11930465, message.positionLong / 11930465])

                        }
                    }

                    decoder.read({
                        mesgListener: onMesg
                    });

                    this.coords = Array<Array<number>>()
                    coords.forEach((coord: number[]) => {
                        this.coords.push(coord)
                    })

                    // Wait for Vue to update
                    this.$nextTick(() => this.$refs.map.leafletObject.fitBounds(this.$refs.polyline.leafletObject.getBounds()))
                };

                reader.readAsArrayBuffer(file);
            }
        }
    }
}

</script>