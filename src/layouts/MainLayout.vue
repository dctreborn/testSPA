<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          Quasar App
        </q-toolbar-title>

        <div>Quasar v{{$q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import axios from 'axios'

export default {
  name: 'MainLayout',

  async mounted () {
    await axios.get('https://mitigation.tti.tamu.edu/wp-json/wp/v2/txdot_roadways')
      .then((response) => {
        console.log(response.data)
      })
  }
}
</script>
