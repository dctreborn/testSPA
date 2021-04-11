<template>
  <div>
    <q-card flat class="row">
      <!-- sorting -->
      <q-card-section class="col-12">
        <!-- item number -->
        <q-btn-group>
          <q-btn-dropdown color="primary" :label="paginationLabel + numResults">
            <q-list separator>
              <q-item v-close-popup v-for="(num, index) in resultsNum" :key="index">
                <q-item-section class="cursor-pointer" @click="changePagination(num)">
                  <q-item-label>{{num}}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </q-btn-dropdown>
          <!-- sort by -->
          <q-btn-dropdown color="secondary" :label="sortLabel + sortByLabel">
            <q-list separator>
              <q-item v-close-popup v-for="(sort, index) in sortBy" :key="index">
                <q-item-section class="cursor-pointer" @click="changeSort(sort)">
                  <q-item-label>{{sort.label}}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </q-btn-dropdown>
          <!-- asc/desc -->
          <q-btn-dropdown color="dark" :label="orderLabel">
            <q-list separator>
              <q-item v-close-popup v-for="(order, index) in orderBy" :key="index">
                <q-item-section class="cursor-pointer" @click="changeOrder(order)">
                  <q-item-label>{{order}}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </q-btn-dropdown>
        </q-btn-group>
      </q-card-section>
      <!-- results -->
      <q-card-section class="col-6">
        Found {{sortedList.length}} results<br>
        <b>Click on item to get more information</b>
        <q-list bordered separator v-if="sortedList.length > 0">
          <q-item clickable v-for="(item, id) in sortedList" :key="id">
            <q-item-section @click="getDetails(item)">
              {{item.title.rendered}}
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
      <!-- details -->
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
      paginationLabel: '# Items: ',
      sortLabel: 'Sort By ',
      sortByLabel: 'Ranking',
      sortKey: 'ranking',
      sortBy: [
        {
          label: 'Ranking',
          key: 'ranking'
        },
        {
          label: 'Congestion Cost',
          key: 'annual_congestion_cost'
        },
        {
          label: 'Hours Delay',
          key: 'annual_hrs_of_delay_per_mile'
        },
        {
          label: 'County',
          key: 'country'
        }
      ],
      orderLabel: 'Desc',
      orderBy: ['Desc', 'Asc'],
      roadworksList: [],
      sortedList: [],
      roadDetails: {},
      metadata: {},
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
    changePagination (num) {
      this.numResults = num
    },

    changeSort (sort) {
      this.sortByLabel = sort.label
      this.sortKey = sort.key
    },

    changeOrder (order) {
      this.orderLabel = order
      this.sortResults()
    },

    sortResults () {
      const tempArr = JSON.parse(JSON.stringify(this.sortedList))
      if (this.orderLabel.toLowerCase() === 'asc') {
        tempArr.sort((a, b) => {
          return a.metadata[this.sortKey] - b.metadata[this.sortKey]
        })
      } else if (this.orderLabel.toLowerCase() === 'desc') {
        tempArr.sort((a, b) => {
          return b.metadata[this.sortKey] - a.metadata[this.sortKey]
        })
      }
      this.sortedList = JSON.parse(JSON.stringify(tempArr))
    },

    async getRoadworks () {
      await axios.get(this.roadAPI + this.pages + this.numResults)
        .then((response) => {
          this.roadworksList = response.data.filter(x => x.status === 'publish')
          this.sortedList = JSON.parse(JSON.stringify(this.roadworksList))
          this.sortResults()
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
