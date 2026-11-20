<template>
    <div class="container-fluid">
        <div class="row">
            <div v-if="!$root.isMobile" class="col-12 col-md-5 col-xl-4">
                <div>
                    <router-link to="/add" class="btn btn-primary mb-3">
                        <font-awesome-icon icon="plus" />
                        {{ $t("Add New Monitor") }}
                    </router-link>
                </div>
                <MonitorList :scrollbar="true" />
            </div>

            <div ref="container" class="col-12 col-md-7 col-xl-8 mb-3">
                <!-- Add :key to disable vue router re-use the same component -->
                <router-view :key="$route.fullPath" :calculatedHeight="height" />
            </div>
        </div>
    </div>
</template>

<script>
import MonitorList from "../components/MonitorList.vue";

export default {
    components: {
        MonitorList,
    },
    data() {
        return {
            height: 0,
        };
    },
    mounted() {
        this.height = this.$refs.container.offsetHeight;
    },
};
</script>

<style lang="scss" scoped>
// The two panes share one wrap: the same content edge on the left and on the
// right at every width, one gutter between them, and both on the 4/8 scale.
// Bootstrap's gutter is a negative margin on the row, which is what made the
// list end 30px from the window while the detail pane ended 14px from it.
.container-fluid {
    width: 100%;
    padding-inline: var(--k-s-4);
}

.container-fluid > .row {
    margin-inline: 0;
    row-gap: var(--k-s-4);
}
</style>
