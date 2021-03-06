<template>
  <sb-render v-if="story && story.content" :item="story.content" />
</template>

<script>
import config from 'config'
import get from 'lodash-es/get'
import { getSettings } from '../helpers'
import StoryblokMixin from '../components/StoryblokMixin'
import { currentStoreView } from '@vue-storefront/core/lib/multistore'

export default {
  name: 'StoryblokPage',
  mixins: [StoryblokMixin],
  metaInfo () {
    if (this.story) {
      const description = get(this.story, 'content.seo.description', get(this.getCurrentStoreView, 'seo.description'))
      return {
        title: get(this.story, 'content.seo.title', this.story.name),
        meta: [
          { vmid: 'description', name: 'description', content: description }
        ],
        link: [
          {
            rel: 'canonical',
            href: this.getCanonical()
          },
          ...this.metaHreflangLinks()
        ]
      }
    }
  },
  computed: {
    getCurrentStoreView () {
      return get(config.storeViews, currentStoreView().storeCode)
    }
  },
  methods: {
    metaHreflangLinks () {
      const { hreflangPrefix } = getSettings(config.storyblok.settings)
      const alternates = get(this.story, 'alternates', [])
      if (hreflangPrefix && alternates.length > 0) {
        const alternateHreflangLinks = this.story.alternates.filter(altStory => {
          const storeCode = this.storeCodeFromSlug(altStory.full_slug)
          return get(config.storeViews, [storeCode, 'disabled'], true) === false
        })
        .map(altStory => {
          const storeCode = this.storeCodeFromSlug(altStory.full_slug)
          const storeView = get(config.storeViews, storeCode)
          return {
            rel: 'alternate',
            hreflang: get(storeView, 'seo.hreflang') || get(storeView, 'i18n.defaultLocale') || storeCode,
            href: this.getCanonical(get(config.storeViews, storeCode), altStory)
          }
        })
        return [
          {
            rel: 'alternate',
            hreflang: get(this.getCurrentStoreView, 'seo.hreflang') || get(this.getCurrentStoreView, 'i18n.defaultLocale') || get(this.getCurrentStoreView, 'i18n.defaultLanguage'),
            href: this.getCanonical()
          },
          ...alternateHreflangLinks
        ]
      } else {
        return []
      }
    },
    getCanonical (storeView = this.getCurrentStoreView, story = this.story) {
      const storeViewUrl = get(storeView, 'url', '')
      const { hreflangPrefix } = getSettings(config.storyblok.settings)
      const url = this.isAbsoluteUrl(storeViewUrl) ? storeViewUrl + '/' + this.removeStoreCodeFromSlug(story.full_slug) : hreflangPrefix + story.full_slug
      return url.replace(/\/home$/, '')
    },
    storeCodeFromSlug (slug) {
      return slug.split(/\/(.+)/)[0]
    },
    removeStoreCodeFromSlug (slug) {
      return slug.split(/\/(.+)/)[1]
    },
    isAbsoluteUrl (url) {
      return /^[a-zA-Z][a-zA-Z\d+\-.]*:/.test(url)
    }
  }
}
</script>
