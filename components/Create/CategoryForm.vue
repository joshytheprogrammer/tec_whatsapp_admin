<template>
  <section>
    <h1 class="subtitle is-size-3">Create new category</h1>
    <form @submit.prevent="submit" class="form">
      <b-field label="Name">
        <b-input v-model="category.name" placeholder="Enter the product name" validation-message="Only letters, numbers and apostrophes are allowed" pattern="^[a-zA-Z 0-9 ']*$" :disabled="loading" required></b-input>
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
export default {
  data() {
    return {
      category: {
        name: '',
        image: '',
      },
      file: {},
      loading: false,
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

      let slug = this.generateSlug()

      await this.$fire.firestore.collection('categories').add({
        name: this.category.name,
        slug: slug,
        active: true,
        thumbnail: this.category.image,
      })
      .then((docRef) => {
        this.$buefy.toast.open({
          duration: 10000,
          message: `Category created with ID - ${docRef.id} `,
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

      let ref = await this.$fire.storage.ref().child('categories/'+this.file.name)

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
        this.category.image = url
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
