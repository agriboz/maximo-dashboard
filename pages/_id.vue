<template>
  <section class="columns">
    <div class="column is-6">
      <h1 class="title">Bekleyen İşler</h1>
      <b-table
        :data="approved"
        :loading="loading"
        :total="totalApproved"
        :per-page="perPage"
        :mobile-cards="false"
        :current-page.sync="currentPageApproved"
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
            label="İş Emri No"
            sortable>
            {{ props.row.wonum }}
          </b-table-column>

          <b-table-column
            field="description"
            label="İsim"
            sortable>
            {{ props.row.description }}
          </b-table-column>

          <b-table-column
            field="statusDateTimeStamp"
            label="Gelme Tarihi ve Saati"
            sortable>
            {{ new Date(props.row.statusDateTimeStamp).toLocaleString('tr') }}
          </b-table-column>

          <b-table-column
            field="workType"
            label="İş Kaynağı">
            {{ props.row.workType }}
          </b-table-column>
        </template>
      </b-table>
    </div>
    <div class="column is-6">
      <h1 class="title">Devam Eden İşler</h1>
      <b-table
        :data="inProgress"
        :loading="loading"
        :total="totalInProgress"
        :per-page="perPage"
        :mobile-cards="false"
        :current-page.sync="currentPageInProgess"
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
            label="İş Emri No"
            sortable>
            {{ props.row.wonum }}
          </b-table-column>

          <b-table-column
            field="description"
            label="İsim"
            sortable>
            {{ props.row.description }}
          </b-table-column>

          <b-table-column
            field="statusDateTimeStamp"
            label="Başlama Tarihi ve Saati"
            sortable>
            {{ new Date(props.row.statusDateTimeStamp).toLocaleString('tr') }}
          </b-table-column>

          <b-table-column
            field="workType"
            label="İş Kaynağı">
            {{ props.row.workType }}
          </b-table-column>
          <b-table-column
            field="percentage"
            label="Std. Süre %Durum">
            {{ secondsToFullHour(props.row.estimatedDuration) }}
            <div class="perc-wrapper">
              <progress
                :value="props.row.percentage"
                :max="100"
                :class="type(props.row.percentage)"
                class="progress"
              > {{ props.row.percentage }}
              </progress>
              <span
                :class="type(props.row.percentage)"
                class="perc-value">
                {{ secondsToFullHour(props.row.timeElapsed) }}
              </span>
            </div>
          </b-table-column>
        </template>
      </b-table>
    </div>
  </section>
</template>

<script>
export default {
  name: 'HomePage',
  data() {
    return {
      data: [],
      isPaginated: true,
      currentPageApproved: 1,
      currentPageInProgess: 1,
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
    inProgress() {
      return this.data.filter(item => item.status === 'INPRG')
    },
    approved() {
      return this.data.filter(item => item.status === 'APPR')
    },

    totalApproved() {
      return this.approved.length
    },

    totalInProgress() {
      return this.inProgress.length
    },

    pageCountApproved() {
      return Math.ceil(this.approved.length / this.perPage)
    },

    pageCountInprogress() {
      return Math.ceil(this.inProgress.length / this.perPage)
    },

    hasLast() {
      return this.currentPage <= this.pageCount - 2
    }
  },

  mounted() {
    this.loadAsyncData()
    setInterval(() => {
      // this.currentPage++

      if (this.currentPageApproved !== this.pageCountApproved) {
        this.currentPageApproved++
      } else {
        this.currentPageApproved = 1
      }
      if (this.currentPageInProgess !== this.pageCountInprogress) {
        this.currentPageInProgess++
      } else {
        this.currentPageInProgess = 1
      }
    }, 5000)
  },

  methods: {
    secondsToFullHour(item) {
      var date = new Date(null)
      date.setSeconds(item)
      const result = date.toISOString().substr(11, 5)
      return result
    },
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

<style scoped>
.perc-wrapper {
  position: relative;
}

.perc-wrapper .perc-value {
  position: absolute;
  position: absolute;
  top: -2px;
  font-size: 0.8rem;
  left: 2px;
}
</style>
