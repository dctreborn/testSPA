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
      <q-card-section class="col-6" v-if="roadDetails.title">
        <q-card flat class="row q-pb-none">
          <q-card-section class="col-12">
            <span class="text-h6">{{metadata.roadway}}</span><br>
            {{metadata.county}} County<br>
            From: {{metadata.from}}<br>
            To: {{metadata.to}}
          </q-card-section>
          <q-card-section class="col-6">
            Annual Hours of Delay per Mile: {{metadata.annual_hrs_of_delay_per_mile}}<br>
            Annual Congestion Cost: {{formatNumber(metadata.annual_congestion_cost)}}<br>
            Year First Entered: {{metadata.year_first_entered}}<br>
            Year Plan Active: {{metadata.year_plan_active}}<br>
            Quarter Plan Active: {{metadata.quarter_plan_active}}
          </q-card-section>
          <q-card-section class="col-6">
            Ranking: {{metadata.ranking}}<br>
            Previous Rankings:
              <span v-for="(data, index) in metadata.previous_ranking" :key="index">
                {{data.year}} (#{{data.rank}})
                <span v-if="index != metadata.previous_ranking.length - 1">
                  , </span>
              </span><br>
          </q-card-section>
          <q-card-section class="col-12">
            Notes: {{metadata.notes || "N/A"}}<br>
            Author: {{roadDetails.authorName}}
          </q-card-section>
        </q-card>
      </q-card-section>
      <q-card-section class="col-6" v-else>
        Nothing to display
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
      metadata: {},
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

    async getDetails (item) {
      this.roadDetails = Object.assign({}, item)
      this.metadata = Object.assign({}, item.metadata)
      await this.getAuthor(item)
    },

    async getAuthor (details) {
      await axios.get(details._links.author[0].href)
        .then((response) => {
          this.$set(this.roadDetails, 'authorName', response.data.name || 'N/A')
        })
    },

    formatNumber (val) {
      return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(val)
    }
  }
}
</script>
