<template>
    <div class="break-text-tab">
        <b-form-group label="Break title" label-for="input-1" description="Use {auto} for dynamic starting/BRB/thanks text">
            <b-form-input id="input-1" v-model="breakTitle" @keydown.ctrl.enter="saveOptions" />
        </b-form-group>


        <div class="d-flex">
            <div class="w-100"></div>
            <b-button variant="success" :disabled="processing || titleProcessing" @click="saveOptions">Save</b-button>
        </div>
    </div>
</template>

<script>
import { authenticatedRequest } from "@/utils/dashboard";
import { unescapeText } from "@/utils/content-utils";

export default {
    name: "BreakTextTab",
    props: { broadcast: Object, titleProcessing: Boolean },
    data: () => ({
        breakTitle: "",
        processing: false
    }),
    computed: {
        broadcastBreakTitle() {
            return unescapeText(this.broadcast?.title);
        }
    },
    methods: {
        async saveOptions() {
            this.processing = true;
            try {
                const data = {
                    title: this.breakTitle
                };

                const response = await authenticatedRequest("actions/update-broadcast", data);
                if (!response.error) {
                    this.$notyf.success(`Break title set to ${data.title}`);
                }
            } finally {
                this.processing = false;
            }
        }
    },
    watch: {
        broadcastBreakTitle: {
            immediate: true,
            handler(newTitle) {
                this.breakTitle = newTitle;
            }
        }
    }
};
</script>

<style scoped>

</style>
