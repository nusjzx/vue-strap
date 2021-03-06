<template>
    <span :class="['card-container', addClass]" ref="cardContainer">
        <div class="morph" v-show="localMinimized">
            <button :class="['morph-display-wrapper', 'btn', btnType, 'card-title']" @click="open()">
                <template v-if="altContent">
                    <div v-html="altContent"></div>
                </template>
                <template v-else>
                    <slot name="header">
                        <div v-html="altContent"></div>
                    </slot>
                </template>
            </button>
        </div>
        <div :class="['card', {'expandable-card': isExpandableCard}, borderType]" v-show="!localMinimized">
            <div :class="['card-header',{'header-toggle':isExpandableCard}, cardType, borderType]"
                 @click.prevent.stop="isExpandableCard && toggle()"
                 @mouseover="onHeaderHover = true" @mouseleave="onHeaderHover = false">
                <div class="header-wrapper" ref="headerWrapper">
                    <slot name="header">
                        <div :class="['card-title', cardType, {'text-white':!isLightBg}]" v-html="headerContent"></div>
                    </slot>
                </div>
                <div class="button-wrapper">
                    <slot name="button">
                        <panel-switch v-show="isExpandableCard && !noSwitchBool && !showCaret" :is-open="localExpanded"
                                      @click.native.stop.prevent="expand()"
                                      @is-open-event="retrieveOnOpen" :is-light-bg="isLightBg"></panel-switch>
                        <button type="button" :class="['close-button', 'btn', isLightBg ? 'btn-outline-secondary' : 'btn-outline-light']"
                                v-show="isSeamless ? onHeaderHover : (!noCloseBool)"
                                @click.stop="close()">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                        </button>
                        <button type="button" :class="['popup-button', 'btn', isLightBg ? 'btn-outline-secondary' : 'btn-outline-light']"
                                v-show="((this.popupUrl !== null) && (!isSeamless || onHeaderHover))"
                                @click.stop="openPopup()">
                            <span class="glyphicon glyphicon-new-window" aria-hidden="true"></span>
                        </button>
                    </slot>
                </div>
            </div>
            <template v-if="preloadBool">
                <div class="card-collapse"
                     ref="panel"
                     v-show="localExpanded"
                >
                    <div class="card-body">
                        <slot></slot>
                        <retriever v-if="hasSrc" ref="retriever" :src="src" :fragment="fragment" delay></retriever>
                        <panel-switch v-show="isExpandableCard && bottomSwitchBool" :is-open="localExpanded"
                                      @click.native.stop.prevent="collapseThenScrollIntoViewIfNeeded()"
                                      @is-open-event="retrieveOnOpen"></panel-switch>
                    </div>
                    <hr v-show="isSeamless" />
                </div>
            </template>
            <template v-else>
                <div class="card-collapse"
                     ref="panel"
                     v-if="localExpanded"
                >
                    <div class="card-body">
                        <slot></slot>
                        <retriever v-if="hasSrc" ref="retriever" :src="src" :fragment="fragment" delay></retriever>
                        <panel-switch v-show="isExpandableCard && bottomSwitchBool" :is-open="localExpanded"
                                      @click.native.stop.prevent="collapseThenScrollIntoViewIfNeeded()"
                                      @is-open-event="retrieveOnOpen"></panel-switch>
                    </div>
                    <hr v-show="isSeamless" />
                </div>
            </template>
        </div>
    </span>
</template>

<script>
  import {getFragmentByHash, toBoolean, toNumber} from './utils/utils.js'
  import md from './utils/markdown.js'
  import panelSwitch from './PanelSwitch.vue'
  import retriever from './Retriever.vue'

  export default {
    components: {
      panelSwitch,
      retriever
    },
    props: {
      header: {
        type: String,
        default: ''
      },
      alt: {
        type: String,
        default: ''
      },
      type: {
        type: String,
        default: null
      },
      expandable: {
        type: Boolean,
        default: true
      },
      isOpen: {
        type: Boolean,
        default: null
      },
      expanded: {
        type: Boolean,
        default: null
      },
      minimized: {
        type: Boolean,
        default: false
      },
      noSwitch: {
        type: Boolean,
        default: false
      },
      noClose: {
        type: Boolean,
        default: false
      },
      popupUrl: {
        type: String,
        default: null
      },
      src: {
        type: String
      },
      bottomSwitch: {
        type: Boolean,
        default: true
      },
      preload: {
        type: Boolean,
        default: false
      },
      addClass: {
        type: String,
        default: ''
      }
    },
    computed: {
      // Vue 2.0 coerce migration
      expandableBool () {
        return toBoolean(this.expandable);
      },
      isOpenBool () {
        return toBoolean(this.isOpen);
      },
      expandedBool () {
        return toBoolean(this.expanded);
      },
      minimizedBool () {
        return toBoolean(this.minimized);
      },
      noSwitchBool () {
        return toBoolean(this.noSwitch);
      },
      noCloseBool () {
        return toBoolean(this.noClose);
      },
      bottomSwitchBool () {
        return toBoolean(this.bottomSwitch);
      },
      preloadBool () {
        return toBoolean(this.preload);
      },
      // Vue 2.0 coerce migration end
      isExpandableCard () {
        return this.expandableBool;
      },
      showCaret () {
        return this.isSeamless;
      },
      isSeamless () {
        return this.type === 'seamless';
      },
      btnType () {
        if (this.isSeamless || this.type === 'light') {
          return 'btn-outline-secondary';
        }
        return 'btn-outline-' + (this.type || 'secondary');
      },
      borderType () {
        if (this.isSeamless) {
          return 'border-0';
        } else if (this.type) {
          if (this.type === 'light') {
            return ''; // Bootstrap 4.x light border is almost invisible on a white page
          }
          return 'border-' + this.type;
        }
        return '';
      },
      cardType () {
        if (this.isSeamless) {
          return 'bg-white';
        }
        return 'bg-' + (this.type || 'light');
      },
      isLightBg () {
        return this.cardType === 'bg-light' || this.cardType === 'bg-white' || this.cardType === 'bg-warning';
      },
      headerContent () {
        return this.renderedHeader;
      },
      altContent () {
        return this.alt && md.render(this.alt) || this.renderedHeader;
      },
      renderedHeader () {
        if (!this.header) {
          return '';
        }

        let htmlRenderedHeader = md.render(this.header).trim();

        if (this.isSeamless) {
          // insert the caret to the header content
          let caretAdded = false;

          // if the header content is wrapped by a <p> or any <h1>, <h2>, ...
          // then it must be inserted inside these HTML tags, otherwise the
          // header content will not be in the same line as caret
          const tags = ['<p>', '<h1>', '<h2>', '<h3>', '<h4>', '<h5>', '<h6>'];
          tags.forEach(tag => {
            if (!caretAdded && htmlRenderedHeader.startsWith(tag))  {
              htmlRenderedHeader = this.insertCaretInsideHeader(htmlRenderedHeader);
              caretAdded = true;
            }
          });

          if (!caretAdded) {
            htmlRenderedHeader = this.caretHtml + ' ' + htmlRenderedHeader;
          }
        }
        return htmlRenderedHeader;
      },
      hasSrc () {
        return this.src && this.src.length > 0;
      },
      showCloseButton () {
        if (!this.isSeamless) {
          return !this.noCloseBool;
        } else {
          return onHeaderHover;
        }
      },
      caretHtml () {
        if (this.localExpanded) {
          return '<span class="glyphicon glyphicon-chevron-down" aria-hidden="true"></span>'
        } else {
          return '<span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>'
        }
      },
      hasCustomHeader () {
        return this.$slots.header !== undefined;
      }
    },
    data () {
      return {
        onHeaderHover: false,
        localExpanded: false,
        localMinimized: false
      }
    },
    methods: {
      toggle() {
        this.localExpanded = !this.localExpanded;
      },
      expand() {
        this.localExpanded = !this.localExpanded;
      },
      close() {
        this.localMinimized = true;
      },
      open() {
        this.localExpanded = true;
        this.localMinimized = false;
      },
      scrollIntoViewIfNeeded() {
        let top = this.$el.getBoundingClientRect().top;
        let isTopInView = (top >= 0) && (top <= window.innerHeight);
        if (!isTopInView) {
          this.$el.scrollIntoView();
        }
      },
      collapseThenScrollIntoViewIfNeeded() {
        this.$once('is-open-event', (el, isOpen) => {
          this.scrollIntoViewIfNeeded();
        });
        this.expand();
      },
      openPopup() {
        window.open(this.popupUrl);
      },
      retrieveOnOpen(el, isOpen) {
        if (isOpen && this.hasSrc) {
          this.$refs.retriever.fetch()
        }
      },
      insertCaretInsideHeader(originalHeaderHTML) {
        const wrappedElementName = jQuery(originalHeaderHTML).attr("name");
        return jQuery(originalHeaderHTML).unwrap().prepend(this.caretHtml + ' ')
            .wrap(`<${wrappedElementName}></${wrappedElementName}>`).parent().html();
      }
    },
    watch: {
      'localExpanded': function (val, oldVal) {
        this.$nextTick(function () {
          this.retrieveOnOpen(this, val);
        })
      },
    },
    created () {
      if (this.src) {
        let hash = getFragmentByHash(this.src);
        if (hash) {
          this.fragment = hash;
          this.src = this.src.split('#')[0];
        }
      }
      // Edge case where user might want non-expandable card that isn't expanded by default
      const notExpandableNoExpand = !this.expandableBool && this.expanded !== 'false';
      // Set local data to computed prop value
      this.localExpanded =  notExpandableNoExpand || this.expandedBool; // Ensure this expr ordering is maintained
      this.localMinimized = this.minimizedBool;
    },
    mounted() {
      this.$nextTick(function () {
        if (this.hasSrc && (this.preloadBool || this.expandedBool)) {
          this.$refs.retriever.fetch()
        }

        if (this.hasCustomHeader) {
          this.$refs.headerWrapper.innerHTML =
            this.insertCaretInsideHeader(this.$refs.headerWrapper.innerHTML);
        }
      });
      const panelHeader = this.$slots.header ? this.$refs.headerWrapper.innerHTML : this.headerContent;
      const panelHeaderText = jQuery(panelHeader).wrap('<div></div>').parent().find(':header').text();
      if (panelHeaderText) {
        this.$refs.cardContainer.setAttribute('id', panelHeaderText.trim().replace(/\s+/g, '-').toLowerCase());
      }
    },
  }
</script>

<style>
    .card-heading {
        width: 100%;
    }

    .card-title {
        display: inline-block;
        font-size: 1em;
        margin: 0;
        vertical-align: middle;
    }

    .card-title * {
        margin: 0px !important;
    }

    .header-wrapper {
        display: inline-block;
        width: 72%;
    }

    .button-wrapper {
        float: right;
        display: inline-block;
        width: 28%;
    }

    .header-toggle {
        cursor: pointer;
    }

    .expandable-card {
        margin-bottom: 0 !important;
        margin-top: 5px;
    }

    .card-group > .card-container > .expandable-card {
        margin-top: 0!important;
    }

    .card-seamless {
        padding: 0;
    }

    .card.card-seamless {
        box-shadow: none;
        border: none;
    }

    .card-seamless > .card-heading {
        padding: 0;
    }

    .card-seamless > .card-collapse > hr {
        margin: 0;
        width: calc(100% - 27px);
    }

    .card-seamless > .card-collapse > .card-body {
        padding: 10px 0;
    }

    .card-seamless > .card-collapse > .card-body > .collapse-button {
        position: relative;
        top: 22px;
    }

    .card-body > .collapse-button {
        margin-bottom: 13px;
        margin-top: 5px;
        opacity: 0.2;
    }

    .card-body > .collapse-button:hover {
        opacity: 1;
    }

    .close-button {
        font-size: 10px !important;
        float: right;
        padding: 3px 8px !important;
        margin-left: 3px;
        margin-top: 2px;
    }

    .popup-button {
        font-size: 10px !important;
        float: right;
        margin-top: 2px;
        padding: 3px 8px !important;
    }

    .morph {
        display: inline-block;
    }

    .morph-display-wrapper {
        white-space: normal;
    }

    /* Bootstrap extra small(xs) responsive breakpoint */
    @media (max-width: 575.98px) {

        .header-wrapper {
            width: 88%;
        }

        .button-wrapper {
            width: 12%;
        }

        .card-body {
            padding: 0.5rem;
        }

        .card-collapse > hr {
            margin-top: 1.5rem;
        }

        .card-header {
            padding: 0.5rem;
        }
    }
</style>
