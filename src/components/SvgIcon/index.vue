<template>
  <svg :class="svgClass" aria-hidden="true" :style="autoStyle">
    <use :xlink:href="iconName"></use>
  </svg>
</template>

<script>
  export default {
    name: 'svg-icon',
    props: {
      iconClass: {
        type: String,
        required: true
      },
      className: {
        type: String
      },
      autoStyle: {
        type: Object
      }
    },
    computed: {
      iconName () {
        return `#icon-${this.iconClass}`
      },
      svgClass () {
        if (this.className) {
          return 'svg-icon ' + this.className
        } else {
          return 'svg-icon'
        }
      }
    }
  }
</script>

<style scoped>
  .svg-icon {
    width: 1em;
    height: 1em;
    padding-left: 2px;
    padding-right: 2px;
    vertical-align: middle;
    fill: currentColor;
    overflow: hidden;
  }
</style>
