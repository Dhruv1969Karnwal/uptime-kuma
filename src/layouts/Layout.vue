<template>
    <div :class="classes">
        <a class="skip-link" href="#app-main">{{ $t("Skip to content") }}</a>

        <div v-if="!$root.socket.connected && !$root.socket.firstConnect" class="lost-connection" role="alert">
            <div class="container-fluid">
                {{ $root.connectionErrorMsg }}
                <div v-if="$root.showReverseProxyGuide">
                    {{ $t("Using a Reverse Proxy?") }}
                    <a href="https://github.com/louislam/uptime-kuma/wiki/Reverse-Proxy" target="_blank">
                        {{ $t("Check how to config it for WebSocket") }}
                    </a>
                </div>
            </div>
        </div>

        <!-- Desktop header -->
        <header v-if="!$root.isMobile" class="topbar">
            <router-link to="/dashboard" class="brand">
                <img class="brand-mark" :src="$root.isDark ? '/icon-dark.svg' : '/icon.svg'" alt="" width="30" height="30" />
                <span class="brand-name">{{ $t("Uptime Kuma") }}</span>
            </router-link>

            <a
                v-if="hasNewVersion"
                target="_blank"
                href="https://github.com/louislam/uptime-kuma/releases"
                class="btn btn-primary btn-sm ms-auto"
            >
                <font-awesome-icon icon="arrow-alt-circle-up" />
                {{ $t("New Update") }}
            </a>

            <ul class="nav nav-pills topnav" :class="{ 'ms-auto': !hasNewVersion }" :aria-label="$t('Main navigation')">
                <li v-if="$root.loggedIn" class="nav-item me-2">
                    <router-link to="/manage-status-page" class="nav-link" :class="{ 'is-current': isCurrent(['/manage-status-page', '/status-page', '/status']) }" :aria-current="isCurrent(['/manage-status-page', '/status-page', '/status']) ? 'page' : undefined">
                        <font-awesome-icon icon="stream" />
                        {{ $t("Status Pages") }}
                    </router-link>
                </li>
                <li v-if="$root.loggedIn" class="nav-item me-2">
                    <router-link to="/dashboard" class="nav-link" :class="{ 'is-current': isCurrent(['/dashboard', '/edit', '/add', '/list']) }" :aria-current="isCurrent(['/dashboard', '/edit', '/add', '/list']) ? 'page' : undefined">
                        <font-awesome-icon icon="tachometer-alt" />
                        {{ $t("Dashboard") }}
                    </router-link>
                </li>
                <li v-if="$root.loggedIn" class="nav-item">
                    <div class="dropdown dropdown-profile-pic">
                        <button
                            type="button"
                            class="profile-btn"
                            data-bs-toggle="dropdown"
                            :aria-label="$t('Account menu')"
                        >
                            <span class="profile-pic" aria-hidden="true">{{ $root.usernameFirstChar }}</span>
                            <font-awesome-icon icon="angle-down" aria-hidden="true" />
                        </button>

                        <!-- Header's Dropdown Menu -->
                        <ul class="dropdown-menu">
                            <!-- Username -->
                            <li>
                                <i18n-t
                                    v-if="$root.username != null"
                                    tag="span"
                                    keypath="signedInDisp"
                                    class="dropdown-item-text"
                                >
                                    <strong>{{ $root.username }}</strong>
                                </i18n-t>
                                <span v-if="$root.username == null" class="dropdown-item-text">
                                    {{ $t("signedInDispDisabled") }}
                                </span>
                            </li>

                            <li><hr class="dropdown-divider" /></li>

                            <!-- Functions -->
                            <li>
                                <router-link
                                    to="/maintenance"
                                    class="dropdown-item"
                                    :class="{ active: $route.path.includes('manage-maintenance') }"
                                >
                                    <font-awesome-icon icon="wrench" />
                                    {{ $t("Maintenance") }}
                                </router-link>
                            </li>

                            <li>
                                <router-link
                                    to="/settings/general"
                                    class="dropdown-item"
                                    :class="{ active: $route.path.includes('settings') }"
                                >
                                    <font-awesome-icon icon="cog" />
                                    {{ $t("Settings") }}
                                </router-link>
                            </li>

                            <li>
                                <a
                                    href="https://github.com/louislam/uptime-kuma/wiki"
                                    class="dropdown-item"
                                    target="_blank"
                                >
                                    <font-awesome-icon icon="info-circle" />
                                    {{ $t("Help") }}
                                </a>
                            </li>

                            <li v-if="$root.loggedIn">
                                <button class="dropdown-item" @click="$root.logout">
                                    <font-awesome-icon icon="sign-out-alt" />
                                    {{ $t("Logout") }}
                                </button>
                            </li>
                        </ul>
                    </div>
                </li>
            </ul>
        </header>

        <!-- Mobile header -->
        <header v-else class="topbar topbar-mobile">
            <router-link to="/dashboard" class="brand">
                <img class="brand-mark" :src="$root.isDark ? '/icon-dark.svg' : '/icon.svg'" alt="" width="28" height="28" />
                <span class="brand-name">{{ $t("Uptime Kuma") }}</span>
            </router-link>
            <a
                v-if="hasNewVersion"
                target="_blank"
                href="https://github.com/louislam/uptime-kuma/releases"
                class="btn btn-primary btn-sm ms-auto"
            >
                {{ $t("New Update") }}
            </a>
        </header>

        <main id="app-main" tabindex="-1">
            <router-view v-if="$root.loggedIn" />
            <Login v-if="!$root.loggedIn && $root.allowLoginDialog" />
        </main>

        <!-- Mobile Only -->
        <div v-if="$root.isMobile" class="bottom-nav-spacer" />
        <nav v-if="$root.isMobile && $root.loggedIn" class="bottom-nav">
            <router-link to="/dashboard" class="nav-link">
                <div><font-awesome-icon icon="tachometer-alt" /></div>
                {{ $t("Home") }}
            </router-link>

            <router-link to="/list" class="nav-link">
                <div><font-awesome-icon icon="list" /></div>
                {{ $t("List") }}
            </router-link>

            <router-link to="/add" class="nav-link">
                <div><font-awesome-icon icon="plus" /></div>
                {{ $t("Add") }}
            </router-link>

            <router-link to="/settings" class="nav-link">
                <div><font-awesome-icon icon="cog" /></div>
                {{ $t("Settings") }}
            </router-link>
        </nav>

        <button
            v-if="numActiveToasts != 0"
            type="button"
            class="btn btn-normal clear-all-toast-btn"
            @click="clearToasts"
        >
            <font-awesome-icon icon="times" />
        </button>
    </div>
</template>

<script>
import Login from "../components/Login.vue";
import compareVersions from "compare-versions";
import { useToast } from "vue-toastification";
const toast = useToast();

export default {
    components: {
        Login,
    },

    data() {
        return {
            toastContainer: null,
            numActiveToasts: 0,
            toastContainerObserver: null,
        };
    },

    computed: {
        // Theme or Mobile
        classes() {
            const classes = {};
            classes[this.$root.theme] = true;
            classes["mobile"] = this.$root.isMobile;
            return classes;
        },

        hasNewVersion() {
            if (this.$root.info.latestVersion && this.$root.info.version) {
                return compareVersions(this.$root.info.latestVersion, this.$root.info.version) >= 1;
            } else {
                return false;
            }
        },
    },

    watch: {},

    mounted() {
        this.toastContainer = document.querySelector(".bottom-right.toast-container");

        // Watch the number of active toasts
        this.toastContainerObserver = new MutationObserver((mutations) => {
            for (const mutation of mutations) {
                if (mutation.type === "childList") {
                    this.numActiveToasts = mutation.target.children.length;
                }
            }
        });

        if (this.toastContainer != null) {
            this.toastContainerObserver.observe(this.toastContainer, { childList: true });
        }
    },

    beforeUnmount() {
        this.toastContainerObserver.disconnect();
    },

    methods: {
        /**
         * Is the current route one of these prefixes? The nav marks the active
         * section, not only an exactly matching path.
         * @param {string[]} prefixes Route prefixes owned by this nav entry
         * @returns {boolean} True when the active route starts with one of them
         */
        isCurrent(prefixes) {
            return prefixes.some((prefix) => this.$route.path.startsWith(prefix));
        },

        /**
         * Clear all toast notifications.
         * @returns {void}
         */
        clearToasts() {
            toast.clear();
        },
    },
};
</script>

<style lang="scss" scoped>
// App shell. Every value comes from the tokens in assets/tokens.scss, so the
// shell themes itself when mixins/theme.js puts `dark` on <body>.
.topbar {
    display: flex;
    align-items: center;
    gap: var(--k-s-3);
    min-height: var(--k-nav-h);
    padding: var(--k-s-2) max(var(--k-gutter), calc((100% - var(--k-wrap)) / 2));
    margin-bottom: var(--k-s-5);
    background-color: var(--k-surface);
    border-bottom: 1px solid var(--k-line);
}

.brand {
    display: flex;
    align-items: center;
    gap: var(--k-s-2);
    min-height: var(--k-row-h);
    padding-right: var(--k-s-2);
    color: var(--k-ink);
    text-decoration: none;

    &:hover .brand-name {
        color: var(--k-accent-strong);
    }
}

.brand-mark {
    flex: none;
    border-radius: var(--k-r-md);
}

.brand-name {
    font-size: var(--k-t-lead);
    font-weight: var(--k-weight-semibold);
    letter-spacing: var(--k-tracking-head);
    color: var(--k-ink);
    transition: color var(--k-dur-in) var(--k-ease);
}

// Nav words: quiet until hovered; the active route is the one filled pill.
.topnav {
    flex: none;
    align-items: center;
    gap: var(--k-s-1);

    .nav-link {
        display: flex;
        align-items: center;
        gap: var(--k-s-2);
        min-height: var(--k-row-h);
        padding: var(--k-s-2) var(--k-s-3);
        font-size: var(--k-t-small);
        font-weight: var(--k-weight-medium);
        color: var(--k-ink-muted);
        text-decoration: none;
        border-radius: var(--k-r-md);

        &:hover {
            color: var(--k-ink);
            background-color: var(--k-surface-hover);
        }

        &.is-current,
        &[aria-current="page"] {
            color: var(--k-on-accent);
            background-color: var(--k-accent);
        }
    }
}

// Profile affordance: a real button (it was a div, so it was unreachable by
// keyboard) that opens the account menu.
.dropdown-profile-pic {
    user-select: none;
}

.profile-btn {
    display: flex;
    align-items: center;
    gap: var(--k-s-2);
    min-height: var(--k-row-h);
    padding: var(--k-s-1) var(--k-s-3) var(--k-s-1) var(--k-s-1);
    color: var(--k-ink);
    cursor: pointer;
    background-color: var(--k-surface-sunken);
    border: 1px solid var(--k-line);
    border-radius: var(--k-r-pill);
    transition:
        background-color var(--k-dur-in) var(--k-ease),
        border-color var(--k-dur-in) var(--k-ease);

    &:hover {
        background-color: var(--k-surface-hover);
        border-color: var(--k-line-strong);
    }
}

.profile-pic {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 28px;
    height: 28px;
    font-family: var(--k-font-mono);
    font-size: var(--k-t-label);
    font-weight: var(--k-weight-medium);
    color: var(--k-on-accent);
    background-color: var(--k-accent);
    border-radius: var(--k-r-pill);
}

main {
    min-height: calc(100vh - var(--k-nav-h) - 96px);
    outline: none;
}

// In flow, not fixed: a banner that floats over the page hides what it reports.
.lost-connection {
    padding: var(--k-s-3) var(--k-s-4);
    font-size: var(--k-t-small);
    color: var(--k-down);
    background-color: var(--k-down-tint);
    border-bottom: 1px solid color-mix(in srgb, var(--k-down-mark) 40%, transparent);
}

// Mobile: a hairline above, no blurred shadow, 56px rows for the thumb.
.bottom-nav-spacer {
    width: 100%;
    height: calc(60px + env(safe-area-inset-bottom));
}

.bottom-nav {
    position: fixed;
    bottom: 0;
    left: 0;
    z-index: 1000;
    display: flex;
    width: 100%;
    height: calc(60px + env(safe-area-inset-bottom));
    padding-bottom: env(safe-area-inset-bottom);
    background-color: var(--k-surface);
    border-top: 1px solid var(--k-line);

    a {
        display: flex;
        flex: 1;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 2px;
        min-height: 56px;
        // four columns at 320px are 80px wide and "Settings" needs 82, so the label
        // wraps onto a second line rather than being cut (the bar has room for it)
        padding-inline: 2px;
        line-height: 1.15;
        font-size: var(--k-t-label);
        color: var(--k-ink-muted);
        text-decoration: none;

        div {
            font-size: 18px;
        }

        &:hover {
            color: var(--k-ink);
        }

        &[aria-current="page"] {
            font-weight: var(--k-weight-medium);
            color: var(--k-accent-strong);
        }
    }
}

.clear-all-toast-btn {
    position: fixed;
    right: var(--k-s-4);
    bottom: var(--k-s-4);
    z-index: 100;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 44px;
    min-height: 44px;
    padding: 0;
    border-radius: var(--k-r-pill);
    box-shadow: var(--k-shadow-pop);
}

@media (max-width: 770px) {
    .topbar {
        margin-bottom: var(--k-s-4);
    }

    .clear-all-toast-btn {
        bottom: 76px;
    }
}
</style>
