<template>
    <div tabindex="-1" class="dropdown" @focusin="openMenu" @focusout="handleFocusOut">
        <button
            type="button"
            class="filter-dropdown-status"
            :class="{ active: filterActive }"
            tabindex="0"
            aria-haspopup="true"
            :aria-expanded="open ? 'true' : 'false'"
            :data-testid="testId"
        >
            <div class="px-1 d-flex align-items-center">
                <slot name="status"></slot>
            </div>
            <span class="px-1">
                <font-awesome-icon icon="angle-down" />
            </span>
        </button>
        <ul class="filter-dropdown-menu" :class="{ open: open }">
            <slot name="dropdown"></slot>
        </ul>
    </div>
</template>

<script>
export default {
    components: {},
    props: {
        filterActive: {
            type: Boolean,
            required: true,
        },
        testId: {
            type: String,
            default: undefined,
        },
    },
    emits: ["openMenu"],
    data() {
        return {
            open: false,
        };
    },
    methods: {
        openMenu() {
            this.$emit("openMenu");
            this.open = true;
        },

        handleFocusOut(e) {
            if (e.relatedTarget != null && this.$el.contains(e.relatedTarget)) {
                return;
            }
            this.open = false;
        },
    },
};
</script>

<style lang="scss">
.filter-dropdown-menu {
    z-index: 100;
    transition: opacity var(--k-dur-panel) var(--k-ease), visibility var(--k-dur-panel) var(--k-ease);
    padding: var(--k-s-1) !important;
    border-radius: var(--k-r-md);
    overflow: hidden;

    position: absolute;
    inset: 0 auto auto 0;
    margin: 0;
    transform: translate(0, 36px);
    box-shadow: var(--k-shadow-pop);
    visibility: hidden;
    list-style: none;
    height: 0;
    opacity: 0;
    color: var(--k-ink);
    background: var(--k-surface);
    border: 1px solid var(--k-line-strong);

    &.open {
        height: unset;
        visibility: inherit;
        opacity: 1;
    }

    .dropdown-item {
        padding: var(--k-s-2) var(--k-s-3);
    }
}

.filter-dropdown-status {
    display: flex;
    align-items: center;
    min-height: 36px;
    padding: var(--k-s-1) var(--k-s-3);
    color: var(--k-ink);
    cursor: pointer;
    background-color: transparent;
    border: 1px solid var(--k-line-strong);
    border-radius: var(--k-r-md);
    transition:
        background-color var(--k-dur-in) var(--k-ease),
        border-color var(--k-dur-in) var(--k-ease);

    &:hover,
    &:focus {
        background-color: var(--k-surface-hover);
    }

    &.active, &.open {
        color: var(--k-ink);
        background-color: var(--k-surface-active);
        border-color: var(--k-accent-strong);
    }
}

.filter-active {
    color: var(--k-accent-strong);
}
</style>
