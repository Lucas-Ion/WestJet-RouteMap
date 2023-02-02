<template>
  <div>
  </div>
  <div class="card dest-list mb-5 mt-5">
    <ul class="list-group list-group-flush">
      <li v-for="dest in dests" v-bind:key="dest.id" class="list-group-item">{{ dest.name }}, {{ dest.countryName }} </li>
    </ul>
  </div>
</template>

<script>
  export default {
    name: "DestList",
    props: {
      dests: Array
    }
  }
</script>

<style lang="scss" scoped>
  .dest-list {
    overflow-y: scroll;
    height: 95vh;
    li {
      &:hover {
        background-color: darkgrey;
      }
    }
  }
</style>


