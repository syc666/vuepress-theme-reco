<template>
  <router-link
    class="nav-link"
    :to="link"
    v-if="!isExternal(link)"
    :exact="exact">
    <reco-icon :icon="`${item.icon}`" />
    {{ item.text }}
  </router-link>
  <a
    v-else
    :href="link"
    class="nav-link external"
    :target="isMailto(link) || isTel(link) ? null : '_blank'"
    :rel="isMailto(link) || isTel(link) ? null : 'noopener noreferrer'"
  >
    <reco-icon :icon="`${item.icon}`" />
    {{ item.text }}
    <OutboundLink/>
  </a>
</template>

<script>
import { defineComponent, computed, toRefs, getCurrentInstance } from 'vue-demi'
import { isExternal, isMailto, isTel, ensureExt } from '@theme/helpers/utils'
import { RecoIcon } from '@vuepress-reco/core/lib/components'

export default defineComponent({
  components: { RecoIcon },

  props: {
    item: {
      required: true
    }
  },

  setup (props, ctx) {
    const instance = getCurrentInstance()

    const { item } = toRefs(props)

    const link = computed(() => ensureExt(item.value.link))

    const exact = computed(() => {
      if (instance.$site.locales) {
        return Object.keys(instance.$site.locales).some(rootLink => rootLink === link.value)
      }
      return link.value === '/'
    })

    return { link, exact, isExternal, isMailto, isTel }
  }
})
</script>
