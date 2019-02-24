<template>
  <section>
    <div class="columns">
      <div
        class="column is-12 has-text-centered has-text-white"
        style="margin-top: 16px; background-color: #2C3A47">
        <h1 class="title  has-text-white is-size-2">{{ ($router.history.current.params.id)?
        ($router.history.current.params.id).toUpperCase():'TÜM İŞLER' }} </h1>
      </div>
    </div>
    <div class="columns">
      <div class="column is-6">
        <h1 class="title  has-text-white is-size-4">Bekleyen İşler</h1>
        <b-table
          :data="approved"
          :loading="loading"
          :total="totalApproved"
          :per-page="perPage"
          :mobile-cards="false"
          :current-page.sync="currentPageApproved"
          :visible="true"
          :paginated="isPaginated"
          :pagination-simple="isPaginationSimple"
          striped>

          <template slot-scope="props">
            <b-table-column
              field="owner"
              label="Sorumlu"
              sortable>
              <div>{{ props.row.owner }}</div>
            </b-table-column>

            <b-table-column
              field="wonum"
              label="İş Emri No"
              sortable>
              <div>{{ props.row.wonum }}</div>
            </b-table-column>

            <b-table-column
              field="description"
              label="İsim"
              sortable>
              <div>{{ props.row.description }}</div>
            </b-table-column>

            <b-table-column
              field="statusDateTimeStamp"
              label="Gelme Tarihi ve Saati"
              sortable>
              <div>{{ new Date(props.row.statusDateTimeStamp).toLocaleString('tr') }}</div>
            </b-table-column>

            <b-table-column
              field="workType"
              label="İş Kaynağı">
              <div>{{ props.row.workType }}</div>
            </b-table-column>
          </template>
        </b-table>
      </div>
      <div class="column is-6">
        <h1 class="title  has-text-white is-size-4">Devam Eden İşler</h1>
        <b-table
          :data="inProgress"
          :loading="loading"
          :total="totalInProgress"
          :per-page="perPage"
          :mobile-cards="false"
          :current-page.sync="currentPageInProgess"
          :visible="true"
          :paginated="isPaginated"
          :pagination-simple="isPaginationSimple"
          striped>

          <template slot-scope="props">
            <b-table-column
              field="owner"
              label="Sorumlu"
              sortable>
              <div>{{ props.row.owner }}</div>
            </b-table-column>

            <b-table-column
              field="wonum"
              label="İş Emri No"
              sortable>
              <div>{{ props.row.wonum }}</div>
            </b-table-column>

            <b-table-column
              field="description"
              label="İsim"
              sortable>
              <div>{{ props.row.description }}</div>
            </b-table-column>

            <b-table-column
              field="statusDateTimeStamp"
              label="Başlama Tarihi ve Saati"
              sortable>
              <div>{{ new Date(props.row.statusDateTimeStamp).toLocaleString('tr') }}</div>
            </b-table-column>

            <b-table-column
              field="workType"
              label="İş Kaynağı">
              <div>{{ props.row.workType }}</div>
            </b-table-column>
            <b-table-column
              field="estimatedDuration"
              label="Std. Süre">
              <div>{{ secondsToFullHour(props.row.estimatedDuration) }}</div>
            </b-table-column>
            <b-table-column
              field="percentage"
              label="Durum">

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
    </div>
    <div class="kayan-yazi">
      <div v-if="k in kayanYaziWrapper">
        <marquee>
          <h6>{{ k.subject }}</h6>
          <p>{{ k.message }}</p>
        </marquee>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: 'HomePage',
  data() {
    return {
      kayanYaziWrapper: [],
      data: [],
      isPaginated: true,
      isPaginationSimple: false,
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

  beforeMount() {
    this.kayanYazi()
  },

  mounted() {
    this.loading = true
    setInterval(() => {
      this.loadAsyncData()
    }, 1000)

    setInterval(() => {
      if (this.currentPageApproved < this.pageCountApproved) {
        this.currentPageApproved++
      } else {
        this.currentPageApproved = 1
      }
      if (this.currentPageInProgess < this.pageCountInprogress) {
        this.currentPageInProgess++
      } else {
        this.currentPageInProgess = 1
      }
    }, 5000)
    this.loading = false
  },

  methods: {
    kayanYazi() {
      this.$axios
        .get(`http://<ip>:9080/maximo-dashboard/api/bulletinboard/`, {
          crossDomain: true
        })
        .then(({ data }) => {
          this.kayanYaziWrapper = data.data
        })
        .catch(error => {
          throw error
        })
    },

    secondsToFullHour(item) {
      var date = new Date(null)
      date.setSeconds(item)
      const result = date.toISOString().substr(11, 5)
      return result
    },
    async loadAsyncData() {
      this.$axios
        .get(
          `http://localhost:9080/maximo-dashboard/api/workorder?type=${
            this.$router.history.current.params.id
          }`,
          { crossDomain: true }
        )
        .then(({ data }) => {
          // api.themoviedb.org manage max 1000 pages
          this.data = data.data
        })
        .catch(error => {
          this.data = []
          this.total = 0
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

<style>
html,
body {
  background-color: #314150;
  height: 100%;
  margin: 0;
}

#__nuxt {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  overflow: auto;
}
#__layout {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  overflow: auto;
}

thead {
  background-color: #51586a;
}

.th-wrap {
  color: white;
  font-size: 14px;
  text-align: left;
}
.table-wrapper {
  height: 90%;
  background: #fafafa;
}

th div {
  height: 50px;
  width: 100px;
  overflow: auto;
}
td div {
  height: 50px;
  width: 100px;
  overflow: auto;
}
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

.pagination-previous {
  display: none;
}
.pagination-next {
  display: none;
}

.scroll-left {
  height: 50px;
  overflow: hidden;
  position: relative;
  background: yellow;
  color: orange;
  border: 1px solid orange;
}
.scroll-left p {
  position: absolute;
  width: 100%;
  height: 100%;
  margin: 0;
  line-height: 50px;
  text-align: center;
  /* Starting position */
  -moz-transform: translateX(100%);
  -webkit-transform: translateX(100%);
  transform: translateX(100%);
  /* Apply animation to this element */
  -moz-animation: scroll-left 15s linear infinite;
  -webkit-animation: scroll-left 15s linear infinite;
  animation: scroll-left 15s linear infinite;
}
/* Move it (define the animation) */
@-moz-keyframes scroll-left {
  0% {
    -moz-transform: translateX(100%);
  }
  100% {
    -moz-transform: translateX(-100%);
  }
}
@-webkit-keyframes scroll-left {
  0% {
    -webkit-transform: translateX(100%);
  }
  100% {
    -webkit-transform: translateX(-100%);
  }
}
@keyframes scroll-left {
  0% {
    -moz-transform: translateX(100%); /* Browser bug fix */
    -webkit-transform: translateX(100%); /* Browser bug fix */
    transform: translateX(100%);
  }
  100% {
    -moz-transform: translateX(-100%); /* Browser bug fix */
    -webkit-transform: translateX(-100%); /* Browser bug fix */
    transform: translateX(-100%);
  }
}

.kayan-yazi {
  position: fixed;
  bottom: 0;
  left: 0;
  height: 50px;
}
</style>
