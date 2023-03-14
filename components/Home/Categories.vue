<template>
  <section>
    <b-table
      :loading="loading"
      :data="data"
      :paginated="isPaginated"
      :per-page="perPage"
      :current-page.sync="currentPage"
      :pagination-simple="isPaginationSimple"
      :pagination-position="paginationPosition"
      :default-sort-direction="defaultSortDirection"
      :pagination-rounded="isPaginationRounded"
      :sort-icon="sortIcon"
      :sort-icon-size="sortIconSize"
      default-sort="name"
      aria-next-label="Next page"
      aria-previous-label="Previous page"
      aria-page-label="Page"
      aria-current-label="Current page"
      :page-input="hasInput"
      :pagination-order="paginationOrder"
      :page-input-position="inputPosition"
      :debounce-page-input="inputDebounce">

      <b-table-column field="id" label="ID" width="40" numeric v-slot="props">
        {{ props.row.id }}
      </b-table-column>

      <b-table-column field="name" label="Name" sortable v-slot="props">
        {{ props.row.name }}
      </b-table-column>

      <b-table-column field="image" label="Image" v-slot="props">
        <img :src="props.row.image" :alt="props.row.name + ' image'">
      </b-table-column>

      <b-table-column field="actions" label="Slug" centered v-slot="props">
        <div class="actions">
          <span><a target="_blank" :href="'https://neasfashion.demo.joshytheprogrammer.com/#'+props.row.id">View</a></span>
        </div>
      </b-table-column>
      <!-- '#'item.id+ -->
    </b-table>
  </section>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      loading: false,
      isPaginated: true,
      isPaginationSimple: false,
      isPaginationRounded: false,
      paginationPosition: 'bottom',
      defaultSortDirection: 'asc',
      sortIcon: 'arrow-up',
      sortIconSize: 'is-small',
      currentPage: 1,
      perPage: 3,
      hasInput: false,
      paginationOrder: '',
      inputPosition: '',
      inputDebounce: ''
    }
  },
  async fetch() {
    try {
      this.loading = true

      await this.$fire.firestore.collection('categories').orderBy('name').get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          this.data.push({id: doc.id, image: doc.data().image, name: doc.data().name})
        })

        this.loading = false
      })
    } catch (error) {
      thus.loading = false
      this.$buefy.toast.open({
        duration: 5000,
        message: `Something went wrong - ${error}`,
        type: 'is-danger'
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.table {
  tr td img {
    object-fit: contain;
    width: 100%;
    max-width: 100px;
    max-height: 100px
  }
}
</style>
