<template>
  <div class="outer" :class="{'full': full}">
    <div class="inner" v-if="!!letter" :class="blockClasses">
      <span>{{letter}}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Block',
  props: ['letter', 'isCursor', 'isCleard', 'full'],
  data () {
    return {
    }
  },
  computed: {
    blockClasses() {
      return {
        full: this.full,
        cursor: this.isCursor,
        cleard: this.isCleard,
      };
    }
  },
  mounted() {
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.outer {
  box-sizing: border-box;
  width: 100%;
  height: 100%;
  position: relative;
}
.outer.full {
  width: 100%;
}
.outer:before {
  content: '';
  display: block;
  padding-top: 100%;
}
.inner {
  position: absolute;
  text-align: center;
  width: calc(100% - 6px);
  height: calc(100% - 6px);
  left: 3px;
  top: 3px;
  padding: 2px 0 0;
  list-style: none;
  background: #557FD9;
  font-weight: bold;
  font-size: 15px;
  box-sizing: border-box;
  display: table;
  overflow: hidden;
}
.inner.cursor {
  background: #BE9D60;
}
.inner.cleard {
  background: #5BCB98;
  transition: all 0.6s ease;
  transform: scale(3);
  opacity: 0;
}
.inner span {
  display: table-cell;
  vertical-align: middle;
}
</style>
