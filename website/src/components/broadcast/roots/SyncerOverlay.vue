<template>
    <div class="syncer-overlay">
        <transition name="fade">
            <IngameOverlay
                v-if="showOverlay"
                id="overlay"
                :broadcast="broadcast"
                :animation-active="true"
                :basic-mode="useBasicOverlay" />
        </transition>
        <transition name="fade">
            <iframe v-show="showSyncer" class="w-100 h-100 position-absolute border-0" :src="syncerURL"></iframe>
        </transition>
    </div>
</template>

<script>
import IngameOverlay from "@/components/broadcast/roots/IngameOverlay";
export default {
    name: "SyncerOverlay",
    components: { IngameOverlay },
    props: ["broadcast", "client"],
    computed: {
        settings() {
            if (!this.broadcast) return;
            return this.broadcast?.observer_settings || [];
        },
        showSyncer() {
            return this.settings.includes("Show syncer");
        },
        showOverlay() {
            return this.settings.includes("Show overlay");
        },
        useBasicOverlay() {
            return this.settings.includes("Use basic overlay");
        },
        syncerURL() {
            let base = "https://syncer.live/?embed&split&noCenter&hideOffset";
            if (this.client?.name) base += `&label=${this.client.name}`;
            return base;
        }
    },
    head() {
        return {
            title: `Syncer | ${this.client?.name || this.broadcast?.code || this.broadcast?.name || ""}`
        };
    }
};
</script>

<style scoped>

</style>
