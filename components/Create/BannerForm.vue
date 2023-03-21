<template>
  <section>
    <h1 class="subtitle is-size-3">Create new banner</h1>
    <form @submit.prevent="submit" class="form">
      <b-field label="Name">
        <b-input v-model="banner.name" placeholder="Enter the banner name" validation-message="Only letters, numbers and apostrophes are allowed" pattern="^[a-zA-Z 0-9 ']*$" :disabled="loading" required></b-input>
      </b-field>
      <b-field label="Device">
        <b-radio v-model="banner.isMobile"
          name="device"
          native-value="true">
          Mobile
        </b-radio>
        <b-radio v-model="banner.isMobile"
          name="device"
          native-value="false">
          Desktop
        </b-radio>
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
      banner: {
        name: '',
        image: '',
        isMobile: false
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

      await this.$fire.firestore.collection('banners').add({
        name: this.banner.name,
        imgLink: this.banner.image,
        isMobile: this.banner.isMobile,
        active: true,
      })
      .then((docRef) => {
        this.$buefy.toast.open({
          duration: 10000,
          message: `Banner created with ID - ${docRef.id} `,
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

      let ref = await this.$fire.storage.ref().child('Banners/'+this.file.name)

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
        this.banner.image = url
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
