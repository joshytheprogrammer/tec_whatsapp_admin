<template>
  <section>
    <b-tabs size="is-medium" position="is-centered" type="is-boxed" v-model="tab">
      <b-tab-item label="Products">

      </b-tab-item>
      <b-tab-item label="Categories">

      </b-tab-item>
      <b-tab-item label="Banners">

      </b-tab-item>
    </b-tabs>
  </section>
</template>

<script>
import Products from "~/components/Home/Products.vue"
import Categories from "~/components/Home/Categories.vue"
export default {
  components: {
    Products,
    Categories
  },
  data() {
    return {
      tab: 0
    }
  },
}
</script>

<style lang="scss" scoped>
section {
  padding: 1rem;
}
</style>
