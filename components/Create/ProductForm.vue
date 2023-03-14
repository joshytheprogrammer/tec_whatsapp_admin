<template>
  <section>
    <h1 class="subtitle is-size-3">Create New Product</h1>
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

      <b-field label="Image">
        <b-field class="file is-primary" :class="{'has-name': !!file}">
          <b-upload v-model="file" accept=".jpg, .JPG, .png, .PNG, .jpeg, .JPEG" class="file-label" validation-message="Only jpg, jpeg and png are allowed" :disabled="loading" required>
            <span class="file-cta">
                <b-icon class="file-icon" icon="upload"></b-icon>
                <span class="file-label">Click to upload</span>
            </span>
            <span class="file-name" v-if="file">
                {{ file.name }}
            </span>
          </b-upload>
        </b-field>
      </b-field>

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
        image: '',
        categories: []
      },
      file: {},
      loading: false,
      catLoading: false,
      allCategories: []
    }
  },
  async mounted() {
    this.catLoading = true

    try {
      await this.$fire.firestore.collection('categories').orderBy('name').get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          this.allCategories.push({id: doc.id, name: doc.data().name})
        })
        this.catLoading = false
      })
    } catch (error) {
      this.catLoading = false
      this.error = error.message
    }

  },
  methods: {
    async submit() {
      // Start Loading animation
      this.loading = true

      // Upload file and check if all is well
      if(!(await this.upload())) {
        this.loading = false
        return
      }

      // Generate product id using random numbers
      let id = this.generateID()

      // Generate product slug
      let slug = this.generateSlug(id, this.product.name)

      await this.$fire.firestore.collection('products').doc(id).set({
        name: this.product.name,
        price: this.product.price,
        image: this.product.image,
        slug: slug,
        categories: this.product.categories
      })
      .then((docRef) => {
        this.$buefy.toast.open({
          duration: 10000,
          message: `Document created successfully with ID - ${id} -  <a style="color: white;" href="https://neasfashion.demo.joshytheprogrammer.com/shop/${slug}" target="_blank" >View</a> `,
          type: 'is-success'
        })

        this.$router.push('/')
      })
      .catch((error) => {
        this.$buefy.toast.open({
          duration: 5000,
          message: `Something went wrong - ${error}`,
          type: 'is-danger'
        })

        this.loading = false
      })
    },
    async upload() {
      // Test file size
      if(this.file.size >= 512000) {
        this.$buefy.toast.open({
          duration: 5000,
          message: `File should be less than 500kb`,
          type: 'is-danger'
        })
        return
      }

      let ref = await this.$fire.storage.ref().child('products/'+this.file.name)

      await ref.put(this.file)
      .then(() => {
        this.$buefy.toast.open({
          duration: 5000,
          message: 'Upload successful',
          type: 'is-success'
        })
      })
      .catch((error) => {
        this.$buefy.toast.open({
          duration: 5000,
          message: `Something went wrong - ${error}`,
          type: 'is-danger'
        })
      })

      await ref.getDownloadURL()
      .then((url) => {
        this.product.image = url
      })
      .catch((error) => {
        this.$buefy.toast.open({
          duration: 5000,
          message: `Something went wrong - ${error.code}`,
          type: 'is-danger'
        })
      })

      return true

    },
    generateID() {
      let name = uuid.v4()

      name = name.slice(0, 8)

      return name
    },
    generateSlug(id, name) {
      name = name + " " + id

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
