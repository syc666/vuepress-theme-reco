<template>
  <ul
    class="sidebar-links"
    v-if="items.length"
  >
    <li v-for="(item, i) in items" :key="i">
      <SidebarGroup
        v-if="item.type === 'group'"
        :item="item"
        :open="i === openGroupIndex"
        :collapsable="item.collapsable || item.collapsible"
        :depth="depth"
        @toggle="toggleGroup(i)"
      />
      <SidebarLink
        v-else
        :sidebarDepth="sidebarDepth"
        :item="item"
      />
    </li>
  </ul>
</template>

<script>
import { defineComponent, ref, getCurrentInstance, toRefs, onUpdated, onMounted } from 'vue-demi'
import SidebarGroup from '@theme/components/SidebarGroup'
import SidebarLink from '@theme/components/SidebarLink'
import { isActive } from '@theme/helpers/utils'

export default defineComponent({
  name: 'SidebarLinks',

  components: { SidebarGroup, SidebarLink },

  props: [
    'items',
    'depth', // depth of current sidebar links
    'sidebarDepth' // depth of headers to be extracted
  ],

  setup (props, ctx) {
    const instance = getCurrentInstance()

    const { items } = toRefs(props)

    const openGroupIndex = ref(0)

    const refreshIndex = () => {
      const index = resolveOpenGroupIndex(
        instance.$route,
        items.value
      )
      if (index > -1) {
        openGroupIndex.value = index
      }
    }

    const activationAnchor = () => {
      // eslint-disable-next-line no-undef
      const anchors = [].slice.call(document.querySelectorAll(AHL_HEADER_ANCHOR_SELECTOR))
        .filter(anchor => decodeURIComponent(instance.$route.fullPath).indexOf(decodeURIComponent(anchor.hash)) != -1)
      if (anchors == null || anchors.length < 1 || anchors[0].offsetTop == undefined) return
      setTimeout(function () {
        window.scrollTo(0, anchors[0].offsetTop + 160)
      }, 100)
    }

    const activationLink = () => {
      const subtitleName = decodeURIComponent(instance.$route.fullPath)
      if (!subtitleName || subtitleName == '') return
      // eslint-disable-next-line no-undef
      const subtitles = [].slice.call(document.querySelectorAll(AHL_SIDEBAR_LINK_SELECTOR))
      for (let i = 0; i < subtitles.length; i++) {
        if (decodeURIComponent(subtitles[i].getAttribute('href')).indexOf(subtitleName) != -1) {
          subtitles[i].click()
          activationAnchor()
          return
        }
      }
    }

    const isInViewPortOfOne = () => {
      const sidebarScroll = document.getElementsByClassName('sidebar')[0]
      let el = document.getElementsByClassName('active sidebar-link')[1]
      if (el == null || el == undefined || el.offsetTop == undefined) {
        el = document.getElementsByClassName('active sidebar-link')[0]
      }
      if (el == null || el == undefined || el.offsetTop == undefined) return

      const viewPortHeight = sidebarScroll.clientHeight || window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight
      const offsetTop = el.offsetTop
      const offsetBottom = el.offsetTop + el.offsetHeight
      const scrollTop = sidebarScroll.scrollTop
      const bottomVisible = (offsetBottom <= viewPortHeight + scrollTop)
      if (!bottomVisible) {
        sidebarScroll.scrollTop = (offsetBottom + 5 - viewPortHeight)
      }
      const topVisible = (offsetTop >= scrollTop)
      if (!topVisible) {
        sidebarScroll.scrollTop = (offsetTop - 5)
      }
    }

    const toggleGroup = (index) => {
      instance.openGroupIndex = index === instance.openGroupIndex ? -1 : index
    }

    const isActive = (page) => {
      return isActive(instance.$route, page.regularPath)
    }

    refreshIndex()

    onMounted(() => {
      activationLink()
      isInViewPortOfOne()
    })

    onUpdated(() => isInViewPortOfOne())

    return { openGroupIndex, refreshIndex, toggleGroup, isActive }
  },

  watch: {
    '$route' () {
      this.refreshIndex()
    }
  }
})

function resolveOpenGroupIndex (route, items) {
  for (let i = 0; i < items.length; i++) {
    const item = items[i]
    if (item.type === 'group' && item.children.some(c => c.type === 'page' && isActive(route, c.path))) {
      return i
    }
  }
  return -1
}
</script>
