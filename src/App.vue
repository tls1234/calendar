<template>
  <div id="app">
    <calendar/>
  </div>
</template>

<script>
import calendar from './components/calendar'

export default {
  name: 'App',
  components: {
    calendar
  }
}
</script>

<style>
@import './reset.css';
</style>
