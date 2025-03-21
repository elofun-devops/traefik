<template>
  <q-header class="shadow-1">
    <section class="app-section bg-primary text-white">
      <div class="app-section-wrap app-boxed app-boxed-xl">
        <q-toolbar class="row no-wrap items-center">
          <div class="q-pr-md logo">
            <img
              alt="logo"
              src="~assets/logo.png">
          </div>
          <q-tabs align="left" inline-label indicator-color="transparent" active-color="white" stretch>
            <q-route-tab to="/" icon="eva-home-outline" no-caps label="Dashboard Elofun" />
            <q-route-tab to="/http" icon="eva-globe-outline" no-caps label="HTTP" />
          </q-tabs>
          <div class="right-menu">
            <q-tabs class="allow-overflow">
              <div
                v-if="!coreVersion.disableDashboardAd && hasHubButtonComponent"
                style="margin-right: 5px;"
              >
                <hub-button-app
                  v-if="$q.dark.isActive"
                  theme="dark"
                />
                <hub-button-app v-if="!$q.dark.isActive" />
              </div>
              <q-btn
                stretch
                flat
                no-caps
                icon="invert_colors"
                :label="themeLabel"
                class="btn-menu"
                @click="cycleTheme"
              />
              <q-btn
                stretch
                flat
                icon="eva-question-mark-circle-outline"
              >
                <q-menu
                  anchor="bottom left"
                  auto-close
                >
                  <q-item>
                    <q-btn
                      type="a"
                      :href="`https://github.com/elofun-devops/traefik`"
                      target="_blank"
                      flat
                      color="accent"
                      align="left"
                      icon="eva-book-open-outline"
                      no-caps
                      label="Documentation"
                      class="btn-submenu full-width"
                    />
                  </q-item>
                  <q-separator />
                  <q-item>
                    <q-btn
                      type="a"
                      href="https://github.com/traefik/traefik/"
                      target="_blank"
                      flat
                      color="accent"
                      align="left"
                      icon="eva-github-outline"
                      no-caps
                      label="GitHub repository"
                      class="btn-submenu full-width"
                    />
                  </q-item>
                </q-menu>
              </q-btn>
            </q-tabs>
          </div>
        </q-toolbar>
      </div>
    </section>

    <section
      class="app-section text-black sub-nav"
      :class="{ 'bg-white': !$q.dark.isActive }"
    >
      <div class="app-section-wrap app-boxed app-boxed-xl">
        <slot />
      </div>
    </section>
  </q-header>
</template>

<script>
import { defineComponent } from 'vue'
import config from '../../../package'
import { mapActions, mapGetters } from 'vuex'

export default defineComponent({
  name: 'NavBar',
  data () {
    return {
      hasHubButtonComponent: false
    }
  },
  computed: {
    ...mapGetters('core', { coreVersion: 'version' }),
    version () {
      if (!this.coreVersion.Version) return null
      return /^(v?\d+\.\d+)/.test(this.coreVersion.Version)
        ? this.coreVersion.Version
        : this.coreVersion.Version.substring(0, 7)
    },
    parsedVersion () {
      if (!this.version) {
        return 'master'
      }
      if (this.version === 'dev') {
        return 'master'
      }
      const matches = this.version.match(/^(v?\d+\.\d+)/)
      return matches ? 'v' + matches[1] : 'master'
    },
    name () {
      return config.productName
    },
    disableDashboardAd () {
      return this.coreVersion.disableDashboardAd
    },
    themeLabel () {
      const mode = this.$q.dark.mode
      if (mode === false) return 'Light theme'
      if (mode === true) return 'Dark theme'
      return 'Auto theme'
    }
  },
  watch: {
    disableDashboardAd (newValue) {
      if (!newValue && customElements.get('hub-button-app') === undefined) {
        const hubButtonScript = document.createElement('script')
        hubButtonScript.async = true
        hubButtonScript.onerror = () => {
          const hubButtonScriptLocal = document.createElement('script')
          hubButtonScriptLocal.async = true
          hubButtonScriptLocal.onload = () => {
            this.hasHubButtonComponent = customElements.get('hub-button-app') !== undefined
          }
          // Sources: https://github.com/traefik/traefiklabs-hub-button-app
          hubButtonScriptLocal.src = 'traefiklabs-hub-button-app/main-v1.js'
          document.head.appendChild(hubButtonScriptLocal)
        }
        hubButtonScript.onload = () => {
          this.hasHubButtonComponent = customElements.get('hub-button-app') !== undefined
        }
        // Sources: https://github.com/traefik/traefiklabs-hub-button-app
      }
    }
  },
  created () {
    this.getVersion()
  },
  methods: {
    ...mapActions('core', { getVersion: 'getVersion' }),
    cycleTheme () {
      const currentMode = this.$q.dark.mode
      let newMode

      if (currentMode === 'auto') newMode = false
      else if (currentMode === false) newMode = true
      else newMode = 'auto'

      this.$q.dark.set(newMode)
      localStorage.setItem('traefik-dark', newMode)
    }
  }
})
</script>

<style scoped lang="scss">
  @import "../../css/sass/variables";

  .q-toolbar {
    min-height: 64px;
  }

  .body--dark {
    .sub-nav {
      background-color: #0e204c;
    }
  }

  .q-item--dark {
    background: var(--q-color-dark);
  }

  .logo {
    display: flex;
    align-items: center;

    img {
      height: 24px;
      margin-right: 10px;
    }

    .version {
      min-height: inherit;
      line-height:  inherit;
      padding: 0 4px;
    }
  }

  .q-tabs {
    color: rgba( $app-text-white, .4 );
    :deep(.q-tabs__content) {
      .q-tab__content{
        min-width: 100%;
        .q-tab__label {
          font-size: 16px;
          font-weight: 600;
        }
      }
    }
  }

  .right-menu {
    flex: 1;
    height: 64px;
    display: flex;
    justify-content: flex-end;
  }

  .btn-menu {
    color: rgba( $app-text-white, .4 );
    font-size: 16px;
    font-weight: 600;
  }

  .q-item {
    padding: 0;
  }

  .btn-submenu {
    font-weight: 700;
    align-items: flex-start;
  }

  .allow-overflow {
    :deep(.q-tabs__content) {
      overflow: visible !important;
    }
  }

</style>
