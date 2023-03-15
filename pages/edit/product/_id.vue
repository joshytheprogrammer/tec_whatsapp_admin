<template>
  <section>
    <h1 class="subtitle is-size-3">Edit This Product</h1>
    <form @submit.prevent="submit" class="form">
      <b-field label="Name">
        <b-input v-model="product.name" placeholder="Enter the product name" validation-message="Only letters, numbers and apostrophes are allowed" pattern="^[a-zA-Z 0-9 ']*$" :disabled="loading" required></b-input>
      </b-field>

      <b-field label="Price">
        <b-input v-model="product.price" placeholder="Enter the product price" validation-message="Only numbers are allowed" pattern="^\d+$" :disabled="loading" required></b-input>
      </b-field>

      <b-field label="Categories">
        <b-select
          multiple
          native-size="2"
          placeholder="Pick this products categories"
          :loading="catLoading"
          required
          v-model="product.categories">
          <option v-for="option in allCategories" :key="option.id" :value="option.id">
            {{option.name}}
          </option>
        </b-select>
      </b-field>

      <p v-if="loading">Fetching product details...</p>

      <b-button native-type="submit" type="is-primary" :loading="loading">Submit</b-button>
    </form>
  </section>
</template>

<script>
import { uuid } from 'vue-uuid'
export default {
  data() {
    return {
      product: {
        name: '',
        price: '',
        categories: []
      },
      loading: false,
      catLoading: false,
      allCategories: []
    }
  },
  async mounted() {
    this.loading = true
    this.catLoading = true

    try {
      await this.$fire.firestore.collection('products').doc(this.$route.params.id).get().then((querySnapshot) => {
        let data = querySnapshot.data()
        this.product.name = data.name
        this.product.price  = data.price
        this.product.categories = data.categories
      })
    }catch(error) {
      this.loading = false
      this.handleError(error)
    }

    try {
      await this.$fire.firestore.collection('categories').orderBy('name').get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          this.allCategories.push({id: doc.id, name: doc.data().name})
        })
        this.catLoading = false
      })
    } catch (error) {
      this.loading = false
      this.catLoading = false
      this.error = error.message
    }


    this.loading = false

  },
  methods: {
    async submit() {
      // Start Loading animation
      this.loading = true
      let slug = this.generateSlug(this.$route.params.id, this.product.name)

      await this.$fire.firestore.collection('products').doc(this.$route.params.id).set({
        name: this.product.name,
        price: this.product.price,
        slug: slug,
        categories: this.product.categories
      }, { merge: true })
      .then(() => {
        this.successMessage('Document changed successfully')
        this.$router.push('/')
      })
      .catch((error) => {
        this.handleError(error)

        this.loading = false
      })
    },
    handleError(error) {
      this.$buefy.toast.open({
        duration: 5000,
        message: `Something went wrong - ${error}`,
        type: 'is-danger'
      })
    },
    successMessage(message) {
      this.$buefy.toast.open({
        duration: 10000,
        message: message,
        type: 'is-success'
      })
    },
    generateSlug(id, name) {
      name = name + " " + id

      name = name.toLowerCase()

      name = name.replace(/\s/g, "-")

      return name
    }
  }
}
</script>

<style lang="scss" scoped>
section {
  padding: 1rem;
  margin: 2rem auto;

  width: 400px;

  display: block;

  box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
  backdrop-filter: blur(2px);

  @media screen and (max-width: $medium) {
    max-width: 90%;
  }
}
</style>
