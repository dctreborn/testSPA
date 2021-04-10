<template>
  <div>
    <q-card flat class="row">
      <q-card-section class="col-12">
        search
      </q-card-section>
      <q-card-section class="col-6">
        list
      </q-card-section>
      <q-card-section class="col-6">
        details
      </q-card-section>
    </q-card>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data () {
    return {
      resultsNum: [10, 50, 100]
    }
  },

  async mounted () {
    await axios.get('https://mitigation.tti.tamu.edu/wp-json/wp/v2/txdot_roadways')
      .then((response) => {
        console.log(response.data)
      })
  }
}
