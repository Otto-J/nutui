<template>
  <div :class="[direction === 'vertical' ? 'vertical-tab' : 'nutui-tab']">
    <div class="tab-title" ref="navlist">
      <div
        :class="['tab-title-box', { 'nut-tab-active': activeIndex == index }]"
        v-for="(item, index) in titles"
        :key="index"
        @click="switchTitle(index, $event)"
      >
        {{ item.title }}
        <TabTitle v-bind:slots="item.content" v-if="item.content"></TabTitle>
      </div>
      <div class="underline"></div>
    </div>
    <div :class="['nutui-tab-swiper', swiperClassName]">
      <div :class="['swiper-wrapper', { 'swiper-no-swiping': noSwiping }]">
        <slot></slot>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
// @ts-nocheck
import { PropType, h, toRefs, reactive, computed, ref, onMounted, nextTick, watch, watchEffect } from 'vue';
import { createComponent } from '@/utils/create';
const { create } = createComponent('tab');
import TabTitle from './tabTitle';
import Swiper from 'swiper';
import 'swiper/swiper-bundle.css';
type TabDirection = 'horizontal' | 'vertical';

export default create({
  props: {
    defaultIndex: {
      type: Number,
      default: 0
    },
    animatedTime: {
      type: Number,
      default: 0
    },
    direction: {
      type: String as PropType<TabDirection>,
      default: 'horizontal'
    },
    noSwiping: {
      type: Boolean,
      default: false
    }
  },
  components: {
    TabTitle
  },
  setup(props, ctx) {
    const titles: any = reactive([]);
    let mySwiper: any = reactive({});
    const isLock = ref(false);
    const activeIndex = ref(props.defaultIndex);
    const navlist: any = ref(null);
    // 生成随机的id
    function createHash() {
      return Array.from(Array(10), () => Math.floor(Math.random() * 36).toString(36)).join('');
    }

    const swiperClassName = ref('swiper-' + createHash());
    //title点击后居中显示
    function centerTitle(index: number) {
      const currEle = navlist.value.querySelectorAll('.tab-title-box')[index];
      if (props.direction === 'vertical') {
        const currTitleTop = navlist.value.offsetTop;
        const currTop = currEle.offsetTop;
        const currHeight = currEle.offsetHeight;
        const tapHeight = navlist.value.offsetHeight;
        navlist.value.scroll(0, currTop - currTitleTop - tapHeight / 2 + currHeight / 2);
      } else {
        const currLeft = currEle.offsetLeft;
        const currWidth = currEle.offsetWidth;
        const tapWidth = navlist.value.offsetWidth;
        navlist.value.scroll(currLeft - tapWidth / 2 + currWidth / 2, 0);
      }
    }

    //切换tab
    function switchTitle(index: number) {
      activeIndex.value = index;
      centerTitle(index);
      mySwiper.slideToLoop(index, props.animatedTime, false);
    }
    function initSwiper(currIndex: number) {
      mySwiper = new Swiper('.' + swiperClassName.value, {
        loop: true /** 循环模式选项 */,
        noSwiping: true,
        observer: true, //修改swiper自己或子元素时，自动初始化swiper
        observeParents: true, //修改swiper的父元素时，自动初始化swiper
        setWrapperSize: true,
        direction: props.direction,
        initialSlide: currIndex,
        on: {
          touchStart: function() {
            isLock.value = true;
          },
          transitionEnd: function(swiper: Swiper): void {
            ctx.emit('switchTab', swiper.realIndex, swiper);
            if (isLock.value) {
              activeIndex.value = swiper.realIndex;
              centerTitle(swiper.realIndex);
            }
          }
        }
      });
    }
    function initTitle() {
      titles.length = 0;
      if (ctx.slots.default) {
        const slots: any[] = ctx.slots.default().length === 1 ? ctx.slots.default()[0].children : ctx.slots.default();
        slots &&
          slots.forEach((item, index) => {
            titles.push({
              title: item.props['tab-title'],
              content: item.children && item.children.header ? item.children.header() : null
            });
          });
      }
      setTimeout(() => {
        initSwiper(activeIndex.value);
      }, 0);
    }
    onMounted(() => {
      initTitle();
    });
    watch(
      () => ctx.slots.default(),
      () => {
        initTitle();
      }
    );
    return {
      swiperClassName,
      titles,
      navlist,
      activeIndex,
      switchTitle
    };
  }
});
</script>

<style lang="scss">
@import 'index.scss';
</style>