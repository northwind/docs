<template>
  <aside class="on-this-page-navigation" :style="[isInConsole && { display: 'block' } ]">
    <div v-show="showOnthisPage">
      <div>
        <ul class="links" v-if="items">
          <OnThisPageItem
            v-for="(link, index) in items"
            :link="link"
            :key="index"
            :activeAnchor="activeAnchor"
          />
        </ul>
        <ul class="links" v-else>
          <OnThisPageItem
            v-for="(link, index) in $page.fullHeaders[0].children"
            :link="link"
            :key="index"
            :activeAnchor="activeAnchor"
          />
        </ul>
      </div>
    </div>
  </aside>
</template>

<script>
import { CONTENT_HEADER_GUTTER } from '@theme/layouts/config'

export default {
  name: 'OnThisPage',
  components: {
    OnThisPageItem: () => import('../components/OnThisPageItem.vue'),
  },
  props: ['items', 'isInConsole'],
  data() {
    return {
      activeAnchor: null,
      anchors: [],
      anchorOffsetPairs: [],
      paddedHeaderHeight: 0,
    }
  },
  computed: {
    showOnthisPage: function() {
      return this.items ||
        (this.$page.fullHeaders[0].children &&
          this.$page.fullHeaders[0].children.length > 0)
        ? true
        : false
    },
  },
  mounted() {
    this.paddedHeaderHeight =
      document.querySelector('.fixed-header').clientHeight +
      CONTENT_HEADER_GUTTER
    this.$nextTick(() => {
      this.captureAnchors()
      this.handleScroll()
      this.setActiveHash()
    })

    window.addEventListener('scroll', this.handleScroll)
    window.addEventListener('scroll', this.setActiveHash)
  },
  updated() {
    this.captureAnchors()
    // this.handleScroll()
    // this.setActiveHash()
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this.handleScroll)
    window.removeEventListener('scroll', this.setActiveHash)
  },
  methods: {
    handleScroll: function(event) {
      // let maxHeight =
      //   document.querySelector('.on-this-page').clientHeight - window.scrollY
      // if (maxHeight > window.innerHeight) {
      //   maxHeight =
      //     window.innerHeight -
      //     document.querySelector('.fixed-header').clientHeight -
      //     CONTENT_HEADER_GUTTER
      // }
      // document.querySelector('.on-this-page-navigation').style.height =
      //   maxHeight - 200 + 'px'
    },

    captureAnchors: function() {
      const sidebarLinks = [].slice.call(
        document.querySelectorAll('.on-this-page-link')
      )
      this.anchors = [].slice
        .call(document.querySelectorAll('.header-anchor'))
        .filter((anchor) =>
          sidebarLinks.some((sidebarLink) => sidebarLink.hash === anchor.hash)
        )
      const anchorOffsets = this.anchors.map(
        (anchor) => anchor.parentElement.offsetTop
      )
      this.anchorOffsetPairs = anchorOffsets.map(
        (anchorOffset, index, anchorOffsets) => [
          anchorOffset,
          anchorOffsets[index + 1],
        ]
      )
    },

    setActiveHash: function(event) {
      if (!this.anchorOffsetPairs.length) {
        return
      }
      const scrollTop = Math.max(
        window.pageYOffset,
        document.documentElement.scrollTop,
        document.body.scrollTop
      )

      const matchingPair =
        scrollTop <= this.anchorOffsetPairs[0][0]
          ? this.anchorOffsetPairs[0]
          : this.anchorOffsetPairs.find(
              (pair) =>
                scrollTop >= pair[0] - this.paddedHeaderHeight &&
                (!pair[1] || scrollTop < pair[1] - this.paddedHeaderHeight),
              this
            )

      const activeAnchor = matchingPair
        ? this.anchors[this.anchorOffsetPairs.indexOf(matchingPair)]
        : this.anchors[0]
      if (activeAnchor && this.activeAnchor !== activeAnchor.hash) {
        this.activeAnchor = activeAnchor.hash
      }
    },
  },
}
</script>

<style lang="stylus">
.on-this-page-navigation
  min-width 180px
  max-width 180px
  flex 0 0 auto
  color #999
  margin-top -12px

.on-this-page
  align-self flex-start
  max-height 'calc(%s - %s - %s)' % (100vh $navbarHeight $headerContentGutter)
  position sticky
  top 0
  overflow-y auto
  top calc(3.6rem + 36px)
  a
    font-size 12px
    border-left 3px solid transparent
    color: #666
    &:visited
      color #666
    &.router-link-active
      border-left-color $accentColor
    &:hover
      color #396aff
    & + ul a
      color: #999
      font-size 12px
      &:visited
        color #999
  // .title
  //   font-size 16px
  //   margin-left 12px
  //   margin-bottom 16px
  //   color #333
  //   font-weight 600
  .links
    border-left 3px solid #eee
    padding-left 16px
  ul
    list-style none
    list-style-type none
    li
      margin-left 0
      margin-bottom 16px
      a
        text-decoration none
        display block
        &.router-link-active
          margin-left -19px
          padding-left 19px
          color $accentColor
      ul
        margin-top 16px
        li
          a
            &.router-link-active
              margin-left -39px
              padding-left 39px

@media screen and (max-width: 1400px) {
  .on-this-page-navigation{
    display none
  }
}

.mobile-on-this-page
  margin-bottom 60px
  h3
    &.mobile-header
      font-size 1rem
  .vs--single
    &.vs--open
      .vs__selected
        position relative
  .vs__dropdown-toggle
    min-height 45px
    padding 0.5rem
    display flex
    align-items center
    justify-content space-between
    flex-wrap nowrap
    .vs__selected-options
      flex-basis auto
      padding 0
      max-width 95%
      .vs__selected
        margin 0
        padding 0
    .vs__actions
      padding 0
    .vs__search
      height 0
      margin 0
      padding 0
  .vs__dropdown-menu
    overflow-x hidden
    border-top-style initial
    z-index 50
    .vs__dropdown-option
      padding 0.7rem 1rem
      border-bottom 1px solid #eee
      width 100%
      .dropdown-item
        width 100%
        overflow hidden
        text-overflow ellipsis
        &.subheading
          margin-left 1rem
      &:last-child
        border-bottom none
  .vs__dropdown-option--highlight
    background inherit
    color inherit
    border 1px solid #99999
</style>
