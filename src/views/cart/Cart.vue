<template>
  <div class="cart">
    <nav-bar class="nav-bar"><div slot="center">购物车({{length}})</div></nav-bar>
    <cart-list/>
    <button-bar/>
  </div>
</template>

<script>
  import NavBar from "@/components/common/navbar/NavBar";
  import CartList from "./childComps/CartList";
  import ButtonBar from "./childComps/ButtonBar";

  import {mapGetters} from 'vuex';
  export default {
    name: "Cart",
    components: {
      NavBar,
      CartList,
      ButtonBar
    },
    computed: {
      ...mapGetters({
        length: 'getCartListLength'
      })
    }
  }
</script>

<style scoped>
  .nav-bar {
    background-color: var(--color-tint);
    color: #fff;
  }
</style>
