<template>
    <transition
            name="zoom"
            @before-enter="beforeEnter"
            @after-enter="afterEnter"
            @before-leave="beforeLeave"
            @after-leave="afterLeave">
        <section class="vm-preview-image preview-image" v-if="isActive">
            <transition name="fade">
                <div class="preview-image-inner" v-show="images.length>0">
                    <p text-center class="info">{{activeIndex + 1}} / {{images.length}}</p>
                    <Slides class="slides"
                            :preloadImages="false"
                            :lazyLoading="true"
                            :initialSlide="activeIndex"
                            :zoom="true"
                            @onClick="onClickHandler"
                            @onSlideChangeEnd="onSlideChangeEndHandler">
                        <Slide class="slide" v-for="(item,index) in images" :key="index">
                            <img :data-src="item" class="swiper-lazy">
                            <div class="swiper-lazy-preloader swiper-lazy-preloader-white"></div>
                        </Slide>
                    </Slides>
                </div>
            </transition>
        </section>
    </transition>
</template>
<style lang="scss" src="./style.scss"></style>
<script type="text/javascript">
  import Slides from '../slides'
  import Slide from '../slide'
  import popupExtend from '../../util/popup-extend'

  export default {
    name: 'PreviewImage',
    extends: popupExtend,
    data () {
      return {
        selected: '',
        images: [],
        activeIndex: 0 // initIndex
      }
    },
    props: {
      urls: {
        type: Array,
        required: true
      },
      current: {
        type: [String, Number],
        required: true
      }
    },
    methods: {
      /**
       * Component Methods
       * */
      onSlideChangeEndHandler (instance) {
        this.activeIndex = instance.activeIndex
        this.selected = this.images[this.activeIndex]
      },
      onClickHandler () {
        this.dismiss()
      }
    },
    created () {
      this.images = this.urls
      this.activeIndex = this.current
      this.selected = this.images[this.activeIndex]
    },
    components: {Slides, Slide}
  }
</script>
