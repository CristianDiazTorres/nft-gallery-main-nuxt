<template>
  <Contact />
</template>

<script lang="ts" >
import { Component, Vue } from 'nuxt-property-decorator'
import Contact from '@/components/landing/Contact.vue'

@Component({
  components: {
    Contact
  }
})

export default class ContactPage extends Vue {
}
</script>
