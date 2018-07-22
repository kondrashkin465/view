<template>
<div class="help-page" v-if="activeQuestion">

  <div class="help-page__title-block">
    <h1 class="help-page__title">{{'help_pg_title' | local}}</h1>
    <Share v-if="!isMobile" />
  </div>

  <div class="layout_between">

    <div class="help-page__right-block">

      <ul v-if="!isMobile" class="help-page__categories">
        <li 
          v-for="category in categoryFaq"
          :class="[{active : activeCategory === category}, 'help-page__categories-item']"
          @click="showCategory(category)"
        >
          {{'faq_category_' + category | local }}
        </li>
      </ul>

      <div class="help-page__support">
        <div class="help-page__support-text">
          <span>{{'no_answer_ask_us' | local}}</span>
        </div>
      </div>

      <a class="help-page__button" href="mailto:support@cub.world">support@cub.world</a>

      <div :class="[{'help-page__status_online':supportOnline}, 'help-page__status']">
        <span class="help-page__status-text">{{'status_support' | local}}</span>
        <span :class="[{free_time: freeTime}, 'help-page__status-value']">
          {{supportOnline ? 'ONLINE':'OFFLINE'}}
        </span>
      </div>

      <div class="help-page__time">
        <span class="help-page__time-text">{{'time_work_supp'|local}}</span>
        <span class="help-page__time-value"> 11.00 - 21.00(GMT +3)</span>
      </div>

    </div>

    <div v-if="!isMobile" class="help-page__list">
      <div 
        v-for="question in questionByCategory(activeCategory)"
        :class="[{active : activeQuestion === question.url }, 'help-page__list-item']"
      >
        <h2 @click="showQuestion(question)" class="help-page__list-item-title">
          {{question.header}}
        </h2>

        <p class="help-page__list-item-content ql-content" v-html="question.content"></p>

      </div>
    </div>

    <div v-if="isMobile" class="help-page__list">
      <div 
        v-for="category in categoryFaq"
        class="help-page__list-category_item"
      >
        <div class="help-page__list-category_name">{{'faq_category_' + category | local }}</div>

        <div 
          v-for="question in questionByCategory(category)"
          :class="[{active : activeQuestion === question.url }, 'help-page__list-item']"
        >

        <h2 @click="showQuestion(question)" class="help-page__list-item-title">
          {{question.header}}
        </h2>

        <p class="help-page__list-item-content ql-content" v-html="question.content" />

      </div>

      </div>
    </div>
    
  </div>

  <Share v-if="isMobile" />

</div>
</template>

<script>
import Share from '@/components/common/Share'
import { uniq } from 'lodash'
import MetaMixin from '@/mixins/metaMixin'

export default {
  name: 'HelpPage',
  components: {
    Share
  },
  mixins: [MetaMixin],
  data () {
    return {
      faqModel: [],
      activeQuestion: null, // url
      activeCategory: 0,
      defaultPageTitle: this.$local('page_help'),
      curDay: new Date().getDay(),
      utcHours: new Date().getUTCHours(),
      isMobile: false
    }
  },
  beforeMount () {
    window.addEventListener('resize', this.handleResize)
  },
  computed: {
    categoryFaq () {
      return uniq(this.faqModel.map(faq => faq.idx)).sort()
    },
    supportOnline () {
      if (this.utcHours >= 8 && this.utcHours < 18) return true
    },
    freeTime () {
      if (this.curDay === 0 || this.curDay === 6) return true
    }
  },
  watch: {
    activeQuestion (url) {
      if (!url) return
      this.pageData = this.faqModel.find(obj => obj.url === url) || {}
      this.activeCategory = this.pageData.idx
    }
  },
  methods: {
    handleResize () {
      this.isMobile = window.offsetWidth < 900
    },
    showCategory (category) {
      if (this.activeCategory === category) { return }
      this.activeCategory = category
      this.activeQuestion = this.faqModel.find(question => question.idx === category).url
      this.$router.replace({
        name: 'HelpOpen',
        params: { question: this.activeQuestion }
      })
    },
    showQuestion (question) {
      if (this.activeQuestion === question.url) { return }
      this.activeQuestion = question.url
      this.$router.replace({
        name: 'HelpOpen',
        params: { question: this.activeQuestion }
      })
    },
    questionByCategory (idx) {
      return this.faqModel.filter(question => question.idx === idx)
    }
  },
  mounted () {
    this.handleResize()
  }
}
</script>

<style lang="scss">
.help-page {
  margin-bottom: 70px;
  &__title-block {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 10px;
    border-bottom: 1px solid $greyL;
    margin-bottom: 40px;
    margin-top: 40px;
  }
  &__list {
    max-width: 700px;
    @media screen and (max-width:1020px) {
      margin-left: 20px;
    }
    &-item {
      margin-bottom: 10px;
      &-title {
        position: relative;
        font-size: 18px;
        font-weight: 400;
        padding: 10px 0 10px 55px;
        background-image: url('~@/assets/icons/common/bg_news_widget.svg');
        &:hover {
          cursor: pointer;
          color: $colorButton;
          &::before {
            background-image: url('~@/assets/icons/common/ico_arrow_blue.svg');
          }
        }
        &::before {
          content: '';
          position: absolute;
          left: 20px;
          top: 50%;
          margin-top: -5px;
          width: 15px;
          height: 10px;
          background-image: url('~@/assets/icons/common/ico_arrow_black.svg');
          background-repeat: no-repeat;
          background-size: 100%;
          transition: 0.4s;
        }
      }
      &-content {
        font-size: 14px;
        max-height: 0;
        overflow: hidden;
        opacity: 0;
        transition: 0.4s;
        @media screen and (max-width:400px) {
          max-width: 300px;
        }
      }
      &.active {
        margin-bottom: 40px;
        .help-page__list-item-content {
          max-height: inherit;
          overflow: visible;
          opacity: 1;
          transition: 0.4s;
        }
        .help-page__list-item-title {
          margin-bottom: 25px;
          &::before {
            transform: rotate(180deg);
            transition: 0.4s;
          }
        }
      }
    }
  }
  &__right-block {
    max-width: 240px;
  }
  &__status {
    font-size: 12px;
    text-align: center;
    &_online {
      .help-page__status-value {
        color: $green;
      }
    }
    &-value {
      color: $red;
    }
  }
  &__time {
    color: $greyL;
    font-size: 12px;
    text-align: center;
    margin-top: 10px;
  }
  &__button {
    height: 45px;
    width: 240px;
    display: inline-block;
    line-height: 43px;
    text-align: center;
    border: 1px solid $colorButton;
    color: $colorButton;
    background-color: transparent;
    background-image: url('~@/assets/icons/common/letter_ico.svg');
    background-repeat: no-repeat;
    background-position: center left 35px;
    background-size: 12px;
    font-size: 14px;
    border-radius: 3px;
    cursor: pointer;
    margin-bottom: 50px;
  }
  &__support-text {
    max-width: 200px;
    color: $purple;
    font-size: 14px;
    text-align: center;
    margin: 30px 0;
    margin-left: auto;
    margin-right: auto;
  }
  &__categories {
    max-width: 240px;
    &-item {
      height: 40px;
      font-size: 16px;
      line-height: 40px;
      cursor: pointer;
      padding-left: 20px;
      &:nth-child(odd) {
        background-color: $grey;
      }
      &:hover,
      &.active {
        background-color: $mainTheme;
        color: $white;
      }
      &.active {
        background-image: url('~@/assets/icons/common/list_active.svg');
        background-size: auto;
        background-repeat: no-repeat;
        background-position: right -1px center;
        &::after {
          content: '';
          display: block;
          position: relative;
          left: 100%;
          border-color: transparent transparent transparent $mainTheme;
          border-style: solid;
          border-width: 20px 0 20px 15px;
          bottom: 40px;
          width: 10px;
        }
      }
    }
  }
  &__title {
    font-size: 28px;
    color: $mainText;
    font-weight: 400;
  }
  @media screen and (max-width:900px) {
    &__list {
      margin-left: 0;
    }
    margin-bottom: 0;
    &__title-block {
      margin-bottom: 30px;
    }
    &__title {
      font-size: 20px;
    }
    .layout_between {
      flex-direction: column;
      align-items: center;
    }
    &__right-block {
      order: 2;
      max-width: 100%;
      margin-bottom: 60px;
    }
    &__support-text {
      margin-top: 0;
      font-size: 20px;
    }
    &__button {
      width: 300px;
      display: block;
      margin-left: auto;
      margin-right: auto;
      font-size: 18px;
      margin-bottom: 40px;
      background-size: 20px;
      background-position-x: 40px;
    }
    &__status {
      font-size: 18px;
    }
    &__time {
      font-size: 14px;
    }
    &__list-item {
      border-bottom: 1px solid rgba(138, 138, 138, 0.3);
      margin-bottom: 0;
      &:last-child {
        margin-bottom: 60px;
      }
      &-title {
        background-image: none;
        color: $colorButton;
        font-size: 16px;
        padding-left: 25px;
        padding-top: 20px;
        padding-bottom: 20px;
        &::before {
          left: 0;
          background-image: url('~@/assets/icons/common/ico_arrow_blue.svg');
        }
      }
      &.active {
        padding-bottom: 40px;
        margin-bottom: 0;
        &:last-child {
          margin-bottom: 60px;
        }
        .help-page__list-item-title {
          margin-bottom: 10px;
        }
      }
    }
    &__list-category_name {
      text-align: center;
      background-color: $grey;
      line-height: 50px;
      margin-bottom: 20px;
      font-size: 18px;
    }
  }
}
</style>
