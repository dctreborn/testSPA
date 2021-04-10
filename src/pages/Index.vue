<template>
  <div>
    <q-card flat class="row">
      <q-card-section class="col-12">
        search
      </q-card-section>
      <q-card-section class="col-6">
        <b>Click on item to get more information</b>
        <q-list bordered separator v-if="roadworksList.length > 0">
          <q-item clickable v-for="(item, id) in roadworksList" :key="id">
            <q-item-section v-if="item.status === 'publish'">
              {{item.title.rendered}}
            </q-item-section>
          </q-item>
        </q-list>
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
      resultsNum: [10, 50, 100],
      roadworksList: [],
      txDotDetails: {},
      regionalDetails: {}
    }
  },

  async mounted () {
    await axios.get('https://mitigation.tti.tamu.edu/wp-json/wp/v2/txdot_roadways')
      .then((response) => {
        this.roadworksList = response.data.map(x => x)
      })
  }
}
</script>
