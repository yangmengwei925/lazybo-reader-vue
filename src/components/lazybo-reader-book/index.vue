<template>
  <div class="lazy-reader" :class="className">
    <!-- 读取文字区域 -->
    <content class="lazy-page-read-content">
      <article class="lazy-read-article" :style="{margin: `0 ${clearance}px`}">
        <section
          ref="readerBook"
          class="lazy-reader-book"
          :style="{transform: `translateX(${translateX}px)`, transition: `all ${animationSeconds}s ease`, fontSize: `${fontSize}rem`}">
          <h3 v-if="title">{{ title }}</h3>
          <p v-for="(item, index) in readerBook" :key="index">{{ item }}</p>
        </section>
      </article>
    </content>
    <!-- 全屏覆盖区域 -->
    <div class="lazy-reader-cover">
      <!-- 上方数据 -->
      <div class="lazy-cover-top">
        <h1 v-if="pageCurrent > 0" class="lazy-read-book-name">
          <slot name="cover-top" :data="slotData">{{ title }}</slot>
        </h1>
      </div>
      <!-- 三块区域 左 中 右 -->
      <div @click="previousPage" class="lazy-cover-left"/>
      <div @click="onCenter" class="lazy-cover-centre"/>
      <div @click="nextPage" class="lazy-cover-right"/>
      <!-- 下方数据 -->
      <div class="lazy-cover-bottom">
        <span class="lazy-page-statistical">
          <slot name="cover-bottom-left" :data="slotData">{{ pageCurrent + 1 }}/{{ pageTotal }}</slot>
        </span>
        <span class="lazy-more">
          <slot name="cover-bottom-right" :data="slotData"/>
        </span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
  import {Component, Emit, Prop, Ref, Vue} from 'vue-property-decorator';

  interface ISlotData {
    page: {
      total: number;
      current: number;
    };
    title: string;
  }

  type toEndType = 0 | 1 | 2 | 3;

  @Component({})
  export default class LazyboReaderBook extends Vue {
    @Prop({default: ''}) public readonly title!: string;
    @Prop({default: ''}) public readonly content!: string;
    @Prop({default: 16}) public readonly clearance!: number;
    @Prop({default: 1.125}) public readonly fontSize!: number;
    @Prop({default: ''}) public readonly className!: string;
    @Prop({default: 0.3}) public readonly animationSeconds!: number;
    @Prop({default: false}) public readonly noLeftRightClick!: boolean;

    @Ref('readerBook') protected readonly readerBookElement!: HTMLElement;

    public pageTotal: number = 0;
    public pageCurrent: number = 0;

    protected translateX: number = 0;

    public previousPage(): void {
      if (this.noLeftRightClick) return;
      if (this.pageCurrent <= 0) {
        this.onToEnd(0);
        return;
      }
      this.pageCurrent--;
      this.calculateTranslateX();
      if (this.pageCurrent - 1 === 0) {
        this.onToEnd(1);
      }
    }
    public nextPage(): void {
      if (this.noLeftRightClick) return;
      if (this.calculatePage(this.pageCurrent)) {
        this.onToEnd(3);
        return;
      }
      this.pageCurrent++;
      this.calculateTranslateX();
      if (this.pageCurrent + 1 === this.pageCurrent) {
        this.onToEnd(2);
      }
    }
    public calculateTotalPage(): void {
      this.$nextTick(() => {
        let pageCount = 0;
        while (!this.calculatePage(pageCount)) {
          pageCount++;
        }
        this.pageTotal = pageCount + 1;
      })
    }

    @Emit()
    public onToEnd(type: toEndType): toEndType {
      return type;
    }

    @Emit()
    public onCenter(): void {
      return;
    }

    get slotData(): ISlotData {
      return {
        page: {
          total: this.pageTotal,
          current: this.pageCurrent + 1
        },
        title: this.title,
      };
    }
    get readerBook(): string[] {
      return this.content.split(/\n/g);
    }

    protected created(): void {
      this.calculateTotalPage();
    }
    protected calculateTranslateX(): void {
      this.translateX = -(this.readerBookElement.offsetWidth + this.clearance) * (this.pageCurrent);
    }
    protected calculatePage(page: number): boolean {
      const number = (this.readerBookElement.offsetWidth) * (page + 1);
      return this.readerBookElement.scrollWidth - number <= this.readerBookElement.offsetWidth;
    }
  }
</script>
<style scoped lang="scss">
  $--content-top: 30px;
  $--content-bottom: 20px;
  $--reader-cover-width: calc(100% / 3);
  $--reader-cover-height: calc(100% - 2rem);

  @mixin absolute-full {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
  }

  .lazy-reader {
    @include absolute-full;
    touch-action: pan-y;
    .lazy-page-statistical {
      font-size: .75rem;
      padding: 1px 16px;
      float: left;
      height: 18px;
      line-height: 18px;
    }
    .lazy-more {
      font-size: .75rem;
      padding: 1px 16px;
      float: right;
      height: 18px;
      line-height: 18px;
    }
    .lazy-read-book-name {
      font-size: .75rem;
      font-weight: 400;
      padding: 2px 18px;
    }
    .lazy-reader-cover {
      @include absolute-full;
      z-index: 1000;
      .lazy-cover-left {
        @include absolute-full;
        width: $--reader-cover-width;
        height: $--reader-cover-height;
        right: unset;
        margin: auto;
      }

      .lazy-cover-centre {
        @include absolute-full;
        left: $--reader-cover-width;
        width: $--reader-cover-width;
        height: $--reader-cover-width;
        right: unset;
        margin: auto;
      }

      .lazy-cover-right {
        @include absolute-full;
        width: $--reader-cover-width;
        height: $--reader-cover-height;
        left: unset;
        margin: auto;
      }

      .lazy-cover-top {
        @include absolute-full;
        bottom: unset;
        height: $--content-top;
      }

      .lazy-cover-bottom {
        @include absolute-full;
        top: unset;
        height: $--content-bottom;
      }
    }

    .lazy-page-read-content {
      position: absolute;
      top: $--content-top;
      bottom: $--content-bottom;
      overflow: hidden;
      border-top: 0;
    }

    .lazy-read-article {
      font-size: 1rem;
      line-height: 1.8;
      overflow: hidden;
      min-height: 100%;
      height: 100%;
      text-align: justify;
    }

    .lazy-reader-book {
      overflow: visible;
      columns: calc(100vw - 32px) 1;
      column-gap: 16px;
      min-height: inherit;
      height: 100%;
      -webkit-columns: calc(100vw - 32px) 1;
      -webkit-column-gap: 16px;
    }

    p {
      word-break: break-all;
      font-size: 1em;
      margin: .1em 0;
    }
  }

  .transitionLeft-enter, .transitionRight-leave-active {
    -webkit-transform: translate(100%, 0);
    transform: translate(100%, 0);
  }

  .transitionLeft-leave-active, .transitionRight-enter {
    -webkit-transform: translate(-100%, 0);
    transform: translate(-100%, 0);
  }
</style>
