<template>
    <div class="container">
        <h2 class="text-center mb-3">Team matches</h2>
        <div class="w-100">
            <ScheduleMatch
                v-for="match in matches"
                :key="match.id"
                :match="match"
                :left-team="team"
                :show-editor-button="showEditorButton" />
        </div>
    </div>
</template>

<script>
import ScheduleMatch from "@/components/website/schedule/ScheduleMatch.vue";
import { ReactiveArray, ReactiveRoot, ReactiveThing } from "@/utils/reactive";
import { sortMatches } from "@/utils/sorts";
import { canEditMatch } from "@/utils/client-action-permissions";
import { useAuthStore } from "@/stores/authStore";

export default {
    name: "TeamMatches",
    components: {
        ScheduleMatch
    },
    props: ["team"],
    computed: {
        matches() {
            if (!this.team?.matches?.length) return [];
            return ReactiveArray("matches", {
                teams: ReactiveArray("teams", {
                    theme: ReactiveThing("theme")
                }),
                maps: ReactiveArray("maps")
            })(this.team).sort(sortMatches);
        },
        hydratedEvent() {
            if (!this.team?.event) return null;
            return ReactiveRoot(this.team.event.id, {
                player_relationships: ReactiveArray("player_relationships")
            });
        },
        showEditorButton() {
            const { user } = useAuthStore();
            return canEditMatch(user, { event: this.hydratedEvent });
        }
    }
};
</script>

<style scoped>

</style>
