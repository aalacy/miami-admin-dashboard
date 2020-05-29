<template>
  <v-container
    id="dashboard"
    fluid
    tag="section"
  >
    <v-card
      icon="mdi-account-multiple"
      title="Owners"
      class="px-5 py-3"
    >
      <v-card-title>
        Dashboard
        <v-spacer></v-spacer>
        <span class="mr-2">Select a Range</span>
          <date-range-picker
            ref="picker"
            v-model="dateRange"
            @update="updateValues"
            :ranges="ranges"
           :locale-data="{ firstDay: 1, format: 'DD-MM-YYYY' }"
          >
            <template v-slot:input="picker" style="min-width: 350px;">
                {{ picker.startDate | date }} - {{ picker.endDate | date }}
            </template>
          </date-range-picker>
          <v-spacer></v-spacer>
      </v-card-title>
      <v-row>
       <v-col
        cols="12"
        sm="6"
        lg="4"
        >
          <base-material-stats-card
            color="success"
            icon="mdi-alert-outline"
            title="Total Signups"
            :value="stats.totalSignups"
            sub-icon="mdi-chevron-triple-right"
            sub-text="Display the total # of teacher signups"
            hover
            ripple
          />
        </v-col>
        <v-col
          cols="12"
          sm="6"
          lg="4"
          >
            <base-material-stats-card
              color="info"
              icon="mdi-alert-outline"
              title="Active Teachers"
              :value="stats.totalActives"
              sub-icon="mdi-chevron-triple-right"
              sub-text="Display the total # of active teacher profiles"
              hover
              ripple
            />
        </v-col>
        <v-col
          cols="12"
          sm="6"
          lg="4"
          >
            <base-material-stats-card
              color="secondary"
              icon="mdi-alert-outline"
              title="Conversion Rate"
              :value="stats.conversionRate"
              sub-icon="mdi-chevron-triple-right"
              sub-text="% of teachers that become active"
              hover
              ripple
            />
        </v-col>
      </v-row>
      <v-row>
        <v-col
          cols="12"
        >
        </v-col>
        <v-col
          cols="12"
          md="12"
        >
          <v-card
            class="pt-3 min-50"
          > 
            <highcharts v-if="data1.length && data2.length"  :updateArgs="[true, false]" :options="comboChart"></highcharts>
          </v-card>
        </v-col>
      </v-row>
    </v-card>
  </v-container>
</template>

<script>
  import { comboChart, getData, getActiveData } from '../../util'
  import axios from 'axios'
  import DateRangePicker from 'vue2-daterange-picker'
  import 'vue2-daterange-picker/dist/vue2-daterange-picker.css'

  export default {
    name: 'DashboardDashboard',

    components: { DateRangePicker  },

    data () {
      let today = new Date()
      today.setHours(0, 0, 0, 0)
      const endDate = new Date(today.getFullYear(), today.getMonth(), today.getDate())
      const startDate = new Date(today.setDate(endDate.getDate() - 500))
      return {
        loading: true,
        menu: false,
        dateRange: {startDate, endDate},
        selectedDates: [],
        totalSignedup: [],
        totalActive: [],
        data1: [],
        data2: [],
        stats: {
          totalSignups: '346',
          totalActives: '302',
          conversionRate: '58%',
        }
      }
    },

    filters: {
      date(val) {
        return val ? val.toLocaleString() : ''
      }
    },

    watch:{
    },

    computed: {
      comboChart () {
        return comboChart(this.data1, this.data2)
      },
      ranges () {
        let today = new Date()
        today.setHours(0, 0, 0, 0)

        let yesterday = new Date()
        yesterday.setDate(today.getDate() - 1)
        yesterday.setHours(0, 0, 0, 0);
        const endDate = new Date(today.getFullYear(), today.getMonth(), today.getDate())
        const startDate = new Date(today.setDate(endDate.getDate() - 500))
        const thisMonthStart = new Date(today.getFullYear(), today.getMonth(), 1)
        const thisMonthEnd = new Date(today.getFullYear(), today.getMonth() + 1, 0)
        const date = new Date()
        const thisWeekEnd = new Date(date.setDate(date.getDate() - (date.getDay() - 1) + 6));
        const lastWeekStart = new Date(date.setDate(thisWeekEnd.getDate() - 13))
        const lastWeekEnd = new Date(date.setDate(thisWeekEnd.getDate() - 7))
        return {
            'Today': [today, today],
            'Yesterday': [yesterday, yesterday],
            'Last Week': [lastWeekStart, lastWeekEnd],
            'This month': [thisMonthStart, thisMonthEnd],
            'Last month': [new Date(today.getFullYear(), today.getMonth() - 1, 1), new Date(today.getFullYear(), today.getMonth(), 0)],
            'This year': [new Date(today.getFullYear(), 0, 1), new Date(today.getFullYear(), 11, 31)],
            'Total': [startDate, endDate]
        }
      },
    },

    mounted () {
      this.totalSignedup = getData(500)
      this.totalActive = getActiveData(this.totalSignedup)

      this.data1 = this.totalSignedup.sort()
      this.data2 = this.totalActive.sort()

      this.calcSummary()
    },

    methods: {
      calcSummary () {
        let totalSignups = 0, totalActives = 0
        this.data1.map(item => totalSignups += item[1])
        this.data2.map(item => totalActives += item[1])

        this.stats.conversionRate = (totalActives / (totalSignups + totalActives) * 100).toFixed(2) + '%'
        this.stats.totalSignups = totalActives.toString()
        this.stats.totalActives = totalActives.toString()
      },
      filterRange (date) {
        this.$refs.menu.save(date)
        const startDate = date[0]
        const endDate = date[1]
        const self = this
        this.data = this.data.filter(item => self.$moment(item.created_at).isSameOrAfter(startDate) && self.$moment(item.created_at).isSameOrBefore(endDate)
        )
      },

      updateValues () {
        const startDate = this.$moment(this.dateRange.startDate).subtract(1, 'days').unix() * 1000
        const endDate = this.$moment(this.dateRange.endDate).unix() * 1000
        this.data1 = this.totalSignedup.filter(item => item[0] >= startDate && item[0] < endDate).sort()
        this.data2 = this.totalActive.filter(item => item[0] >= startDate && item[0] < endDate).sort()

        this.calcSummary()
      }
    },
  }
</script>

<style>
</style>