<template>
    <div class="caster-background">
        <transition name="map-fade" mode="in-out">
            <div v-if="mapToShow" :key="mapToShow.id" class="map w-100 h-100">
                <div class="map-image w-100 h-100" :style="mapImage(mapToShow)">
                    <video
                        v-if="useVideo && mapVideo(mapToShow)"
                        class="map-video w-100 h-100"
                        :src="mapVideo(mapToShow)"
                        autoplay
                        muted
                        loop></video>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
import { ReactiveArray, ReactiveRoot, ReactiveThing } from "@/utils/reactive";
import { bg, getNewURL } from "@/utils/images";

function mapCheck(map, search) {
    if (!map.name) return false;
    if (map.name.toLowerCase() === search) return true;
    if (map.name.indexOf(" ") !== -1 && map.name.split(" ")[0].toLowerCase() === search) return true;
    if (map.name.toLowerCase().includes(search.toLowerCase())) return true;
}

export default {
    name: "CasterBackground",
    props: ["broadcast", "defaultMap", "useVideo"],
    computed: {
        liveMatch() {
            if (!this.broadcast?.live_match) return {};
            return ReactiveThing("live_match", {
                maps: ReactiveArray("maps", {
                    map: ReactiveThing("map")
                })
            })(this.broadcast);
        },
        maps() {
            return this.liveMatch?.maps || [];
        },
        nextMap() {
            const unplayedMaps = this.maps.filter(m => !m.dummy && !m.winner && !m.draw);
            return unplayedMaps[0]?.map;
        },
        allMaps() {
            return ReactiveRoot("Map Data", {
                ids: ReactiveArray("ids", {
                    map: ReactiveThing("map")
                })
            })?.ids || [];
        },
        requestedMap() {
            if (!this.allMaps?.length) return null;

            let req;
            if (this.defaultMap) req = this.allMaps.find(map => mapCheck(map, this.defaultMap));
            // if (!req) req = this.allMaps.find(map => map.name === "King's Row");
            if (!req) {
                // get a random one
                const maps = this.allMaps.filter(map => map.type && map.type.indexOf("(") === -1 && map.video);
                req = maps[Math.floor(Math.random() * maps.length)];
            }
            return req;
        },
        mapToShow() {
            return this.nextMap || this.requestedMap || null;
        }
    },
    methods: {
        mapImage(map) {
            console.log(map);
            return bg((map.big_image || map.image)?.[0].url);
        },
        mapVideo(map) {
            const video = map?.video?.[0];
            if (!video) return null;
            return getNewURL(video, "orig");
        }
    },
    head() {
        return {
            title: `Caster Background | ${this.broadcast?.code || this.broadcast?.name || ""}`
        };
    }
};
</script>

<style scoped>
    .caster-background {
        width: 100vw;
        height: 100vh;
        overflow: hidden;
    }
    .map {
        position: absolute;
        top: 0;
        left: 0;
    }
    .map-image {
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
    }
    .map-video {
        object-fit: cover;
    }


    .map-fade-enter-active, .map-fade-leave-active { transition: opacity 250ms ease 2s; }
    .map-fade-enter-from, .map-fade-leave-to { opacity: 0; }
    .map-fade-enter-to, .map-fade-leave-from { opacity: 1; }
</style>
