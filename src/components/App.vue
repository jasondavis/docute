<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
  import {mapState, mapActions} from 'vuex'
  import throttle from 'lodash.throttle'
  import {findMax, findMin} from 'utils'
  import {$$} from 'utils/dom'

  export default {
    name: 'app',
    computed: {
      ...mapState(['jumping'])
    },
    mounted() {
      this.scrollSpy()
      this.detectClick()
    },
    methods: {
      ...mapActions(['updateActiveId', 'jumpToId']),
      scrollSpy() {
        const handleScroll = () => {
          const name = this.$route.meta && this.$route.meta.name
          const isDocPage = ['home', 'page'].indexOf(name) > -1
          const headings = $$('.markdown-toc-heading')
          if (this.jumping || !isDocPage || headings.length === 0) return
          const els = [...headings].map(heading => {
            return {
              top: heading.getBoundingClientRect().top,
              id: heading.id
            }
          })
          const lastNegative = findMax(els.filter(el => el.top < 0), 'top')[0]
          const firstPositive = findMin(els.filter(el => el.top > 0), 'top')[0]

          let el = {}
          if (lastNegative && firstPositive && firstPositive.top > 200) {
            el = lastNegative
          } else if (firstPositive) {
            el = firstPositive
          } else {
            el = els[els.length - 1]
          }
          if (el.id) this.updateActiveId(el.id)
        }

        document.addEventListener('scroll', throttle(handleScroll, 300))
      },
      detectClick() {
        document.addEventListener('click', e => {
          this.handleNavigateAttribute(e)
        })
      },
      handleNavigateAttribute(e) {
        const jumpEl = e.target.closest('[jump-to-id]')
        const id = jumpEl && jumpEl.getAttribute('jump-to-id')
        if (id) {
          e.preventDefault()
          this.$router.push({query: {...this.$route.query, id}})
          return this.jumpToId(id)
        }

        const linkEl = e.target.closest('[router-link]')
        const link = linkEl && linkEl.getAttribute('router-link')
        if (link) {
          e.preventDefault()
          return this.$router.push(link)
        }
      }
    }
  }
</script>

<style src="css/main.css"></style>
