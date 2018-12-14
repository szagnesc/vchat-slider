
<template>
  <div class="ax-swiper-item">
    <slot></slot>
  </div>
</template>

<script type="text/babel">
export default {
  name: 'ax-swiper-item',
  mounted () {
    this.$nextTick(this.$parent.init)
  }
}
</script>
