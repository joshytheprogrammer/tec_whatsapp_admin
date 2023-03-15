<template>
  <section>
    <h1 class="subtitle is-size-3">Edit this category</h1>
    <form @submit.prevent="submit" class="form">
      <b-field label="Name">
        <b-input v-model="category.name" placeholder="Enter the product name" validation-message="Only letters, numbers and apostrophes are allowed" pattern="^[a-zA-Z 0-9 ']*$" :disabled="loading" required></b-input>
      </b-field>

      <p v-if="loading">Fetching category details...</p>

      <b-button native-type="submit" type="is-primary" :loading="loading">Submit</b-button>
    </form>
  </section>
</template>

<script>
export default {
  data() {
    return {
      category: {
        name: '',
      },
      loading: false,
    }
  },
  async mounted() {
    this.loading = true

    try {
      await this.$fire.firestore.collection('categories').doc(this.$route.params.id).get().then((querySnapshot) => {
        let data = querySnapshot.data()
        this.category.name = data.name
      })
    }catch(error) {
      this.loading = false
      this.handleError(error)
    }

    this.loading = false
  },
  methods: {
    async submit() {
      // Start Loading animation
      this.loading = true

      let slug = this.generateSlug()

      await this.$fire.firestore.collection('categories').doc(this.$route.params.id).set({
        name: this.category.name,
        slug: slug,
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
    generateSlug() {
      let name = this.category.name.replace(/\s/g, "-")

      name = name.toLowerCase()

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
