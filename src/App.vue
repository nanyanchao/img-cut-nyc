<template>
    <ImgCut v-bind="$attrs">
      <slot />
    </ImgCut>
</template>

<script>
import ImgCut from './components/imgCut'

export default {
  name: 'app',
  components: {
    ImgCut
  }
}
</script>

<style>

</style>
