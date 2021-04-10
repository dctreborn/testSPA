<template>
  <div>
    <q-card flat class="row">
      <q-card-section class="col-12">
        <q-select
          style="width: 150px"
          v-model="numResults"
          :options="resultsNum"
          label="Number of Results"
        />
      </q-card-section>
      <q-card-section class="col-6">
        Found {{roadworksList.length}} results<br>
        <b>Click on item to get more information</b>
        <q-list bordered separator v-if="roadworksList.length > 0">
          <q-item clickable v-for="(item, id) in roadworksList" :key="id">
            <q-item-section @click="getDetails(item)">
              {{item.title.rendered}}
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
      <q-card-section class="col-6">
        Details<br>
        {{roadDetails}}
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
      numResults: 10,
      roadworksList: [],
      roadDetails: {},
      txDotDetails: {},
      regionalDetails: {},
      roadAPI: 'https://mitigation.tti.tamu.edu/wp-json/wp/v2/txdot_roadways',
      pages: '?per_page='
    }
  },

  async mounted () {
    this.getRoadworks()
  },

  watch: {
    numResults () {
      this.getRoadworks()
    }
  },

  methods: {
    async getRoadworks () {
      await axios.get(this.roadAPI + this.pages + this.numResults)
        .then((response) => {
          this.roadworksList = response.data.filter(x => x.status === 'publish')
        })
    },

    getDetails (item) {
      this.roadDetails = Object.assign({}, item)
    }
  }
}
</script>
