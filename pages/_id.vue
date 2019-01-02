<template>
  <section class="section">
    <b-table
      :data="data"
      :loading="loading"
      :total="total"
      :per-page="perPage"
      :mobile-cards="false"
      :current-page.sync="currentPage"
      :visible="true"
      :paginated="isPaginated"
      striped>

      <template slot-scope="props">
        <b-table-column
          field="owner"
          label="Sorumlu"
          sortable>
          {{ props.row.owner }}
        </b-table-column>

        <b-table-column
          field="wonum"
          label="Is Emri No"
          sortable>
          {{ props.row.wonum }}
        </b-table-column>

        <b-table-column
          field="description"
          label="Isim"
          sortable>
          {{ props.row.description }}
        </b-table-column>

        <b-table-column
          field="statusDateTimeStamp"
          label="Release Date"
          sortable>
          {{ new Date(props.row.statusDateTimeStamp).toLocaleDateString('tr') }}
        </b-table-column>

        <b-table-column
          field="workType"
          label="Is Kaynagi">
          {{ props.row.workType }}
        </b-table-column>
        <b-table-column
          field="estimatedDuration"
          label="Std Süre">
          {{ props.row.estimatedDuration }}
        </b-table-column>
        <b-table-column
          field="percentage"
          label="Durum">
          <progress
            :value="props.row.percentage"
            :max="100"
            :class="type(props.row.percentage)"
            class="progress"
          > {{ props.row.percentage }}
          </progress>
          <span
            :class="type(props.row.percentage)"
            class="tag">
            {{ props.row.percentage }}
          </span>
        </b-table-column>
      </template>
    </b-table>

  </section>
</template>

<script>
export default {
  /* async asyncData({ $axios, params }) {
    const { data } = await $axios.get(`http://localhost:3001/${params.id}`)
    return { data }
  }, */
  name: 'HomePage',
  data() {
    return {
      data: [],
      isPaginated: true,
      currentPage: 1,
      total: 0,
      loading: false,
      sortField: 'workorderId',
      sortOrder: 'desc',
      defaultSortOrder: 'desc',
      page: 1,
      perPage: 10
    }
  },
  computed: {
    pageCount() {
      return Math.ceil(this.data.length / this.perPage)
    },
    hasLast() {
      return this.currentPage <= this.pageCount - 2
    }
  },

  mounted() {
    this.loadAsyncData()
    setInterval(() => {
      // this.currentPage++

      if (this.currentPage !== this.pageCount) {
        this.currentPage++
      } else {
        this.currentPage = 1
      }
    }, 5000)
  },

  methods: {
    async loadAsyncData() {
      console.log(this.$router)
      this.loading = true
      this.$axios
        .get(`http://localhost:3001/${this.$router.history.current.params.id}`)
        .then(({ data }) => {
          console.log(data)
          // api.themoviedb.org manage max 1000 pages
          this.data = data
          this.loading = false
        })
        .catch(error => {
          this.data = []
          this.total = 0
          this.loading = false
          throw error
        })
    },
    type(value) {
      const number = parseFloat(value)
      if (number > 80) {
        return 'is-danger'
      } else if (number >= 59 && number < 79) {
        return 'is-warning'
      } else if (number <= 58) {
        return 'is-success'
      }
    }
  }
}
</script>
