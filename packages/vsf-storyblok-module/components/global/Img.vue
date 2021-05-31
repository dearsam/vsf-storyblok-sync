<template>
  <div v-if="div && lazy" class="bg">
    <img
      class="lazyload"
      :src="imageScaled(1)"
      :srcset="placeholderSrc"
      :data-srcset="srcSet"
      :width="intrinsicWidth"
      :height="intrinsicHeight"
      decoding="async"
    >
    <slot />
  </div>
  <div v-else-if="div" class="bg">
    <picture>
      <source type="image/webp" :srcset="srcSetWebp" :sizes="sizes">
      <img
        :src="imageScaled(1)"
        :srcset="srcSet"
        :sizes="sizes"
        :width="intrinsicWidth"
        :height="intrinsicHeight"
        decoding="async"
      >
    </picture>
    <div class="slot">
      <slot />
    </div>
  </div>
  <picture v-else-if="lazy">
    <source type="image/webp" :data-srcset="srcSetWebp" :sizes="sizes">
    <img
      class="lazyload"
      :src="imageScaled(1)"
      :srcset="placeholderSrc"
      :data-srcset="srcSet"
      :sizes="sizes"
      :width="intrinsicWidth"
      :height="intrinsicHeight"
      decoding="async"
    >
  </picture>
  <picture v-else>
    <source type="image/webp" :srcset="srcSetWebp" :sizes="sizes">
    <img
      :src="imageScaled(1)"
      :srcset="srcSet"
      :sizes="sizes"
      :width="intrinsicWidth"
      :height="intrinsicHeight"
      decoding="async"
    >
  </picture>
</template>

<script>
import get from 'lodash-es/get'
import config from 'config'
import { mapGetters } from 'vuex'

export default {
  name: 'StoryblokImage',
  props: {
    placeholder: {
      type: String,
      default: ''
    },
    detectWebp: {
      type: Boolean,
      default: get(config, 'storyblok.imageService.defaultWebp', true)
    },
    height: {
      type: Number,
      default: 0
    },
    width: {
      type: Number,
      default: 0
    },
    src: {
      type: String,
      required: true
    },
    div: {
      type: Boolean,
      default: false
    },
    lazy: {
      type: Boolean,
      default: true
    },
    smart: {
      type: Boolean,
      default: false
    },
    fitIn: {
      type: Boolean,
      default: false
    },
    filters: {
      type: Array,
      default: () => []
    },
    sizes: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    ...mapGetters({
      supportsWebp: 'storyblok/supportsWebp'
    }),
    computedFilters () {
      if (this.detectWebp && this.supportsWebp) {
        return [...this.filters, 'format(webp)']
      }
      return this.filters
    },
    image () {
      if (!this.src.includes('//a.storyblok.com')) {
        return this.src
      }
      const [, resource] = this.src.split('//a.storyblok.com')
      let mod = ''
      if (this.height > 0 || this.width > 0) {
        if (this.fitIn) {
          mod += '/fit-in'
        }
        mod += `/${this.width}x${this.height}`
        if (this.smart) {
          mod += '/smart'
        }
      }
      if (this.computedFilters.length) {
        mod += '/filters:' + this.computedFilters.join(':')
      }
      return 'https://img2.storyblok.com' + mod + resource
    },
    srcSet () {
      return [
        `${this.imageScaled(1 / 3)} ${this.intrinsicWidth / 3}w`,
        `${this.imageScaled(1 / 2)} ${this.intrinsicWidth / 2}w`,
        `${this.imageScaled(1)} ${this.intrinsicWidth}w`
      ].join(',')
    },
    srcSetWebp () {
      return [
        `${this.imageScaled(1 / 3, 'webp')} ${this.intrinsicWidth / 3}w`,
        `${this.imageScaled(1 / 2, 'webp')} ${this.intrinsicWidth / 2}w`,
        `${this.imageScaled(1, 'webp')} ${this.intrinsicWidth}w`
      ].join(',')
    },
    intrinsicWidth () {
      return this.intrinsicSize?.width
    },
    intrinsicHeight () {
      return this.intrinsicSize?.height
    },
    intrinsicSize () {
      try {
        const widthHeight = this.image.match(/\d+x\d+/g)[0].split('x')
        return {
          width: widthHeight[0],
          height: widthHeight[1]
        }
      } catch (e) {
        return undefined
      }
    },
    placeholderSrc () {
      return this.placeholder || 'data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==' // `data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 ${this.intrinsicWidth} ${this.intrinsicHeight}"%3E%3C/svg%3E`
    }
  },
  methods: {
    imageScaled (factor, format) {
      if (!this.src.includes('//a.storyblok.com')) {
        return this.src
      }
      const [, resource] = this.src.split('//a.storyblok.com')
      let mod = ''
      if (factor !== 1) {
        if (this.fitIn) {
          mod += '/fit-in'
        }
        mod += `/${Math.round(this.intrinsicWidth * factor)}x${Math.round(this.intrinsicHeight * factor)}`
        if (this.smart) {
          mod += '/smart'
        }
      }
      if (format === 'webp') {
        mod += '/filters:format(webp)'
      }
      return 'https://img2.storyblok.com' + mod + resource
    }
  }
}
</script>

<style lang="scss" scoped>
  img {
    height: auto;
  }
  .bg {
    overflow: hidden;
    position: relative;
    img {
      position: absolute;
      top: 0;
      left: 0;
      object-fit: cover;
      height: 100%;
      width: 100%;
      top: 50% !important;
      transform: translate(0,-50%);
      mix-blend-mode: multiply;
    }
  }
  .slot{
    position: relative;
  }
</style>
