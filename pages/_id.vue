<template>
  <section id="main-content">
    <div
      id="header">
      <h1 class="title has-text-centered has-text-white">{{ ($router.history.current.query.type)?
      ($router.history.current.query.type).toUpperCase():'TÜM İŞLER' }} </h1>
    </div>
    <div
      id="content"
      class="columns">
      <div class="column is-6">
        <h1 class="table_label title  has-text-white is-size-4">Bekleyen İşler</h1>
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
              <div>{{ timestampToDateString(props.row.statusDateTimeStamp) }}</div>
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
        <h1 class="table_label title  has-text-white is-size-4">Devam Eden İşler</h1>
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
              <div>{{ timestampToDateString(props.row.statusDateTimeStamp) }}</div>
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

              <div
                v-if="props.row.percentage"
                class="perc-wrapper">
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
    <div
      v-if="bbmessage.length>0"
      id="footer">
      <div>
        <marquee
          class="marquee"
          direction="left"
          scrollamount="11"
          behavior="scroll">
          <p>{{ bbmessage }}</p>
        </marquee>
      </div>
    </div>
  </section>
</template>

<script>
import axios from 'axios'

export default {
  /* async asyncData({ query, params }) {
    console.log(query, params)
    // We can use async/await ES6 feature
    const { data } = await axios
      .get(`/api/workorder?type=${query.type}`, {
        crossDomain: true
      })
      .catch(error => {
        this.data = []
        this.total = 0
        throw error
      })
    return { data: data }
  }, */

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
    bbmessage() {
      var messages = this.kayanYaziWrapper.map(function(m) {
        return m.message
      })

      var message = messages.join()
      var div = document.createElement('div')
      div.innerHTML = message
      var text = div.textContent || div.innerText || ''
      return text
    },
    inProgress() {
      var fData = this.data.filter(item => item.status === 'INPRG') || []
      if (fData.length % this.perPage > 0) {
        for (let i = fData.length % this.perPage; i < this.perPage; i++) {
          fData.push({})
        }
      }
      return fData
    },
    approved() {
      var fData = this.data.filter(item => item.status === 'APPR') || []
      if (fData.length % this.perPage > 0) {
        for (let i = fData.length % this.perPage; i < this.perPage; i++) {
          fData.push({})
        }
      }
      return fData
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
    var currentUrl = 'http://10.10.10.18:9080/maximo-dashboard/'
    var appName = 'maximo-dashboard'
    var appNameIndex = currentUrl.lastIndexOf(appName)
    if (appNameIndex > 0) {
      currentUrl = currentUrl.substring(0, appNameIndex + appName.length)
    }
    this.$axios.defaults.baseURL = currentUrl

    this.loading = true
    setInterval(() => {
      this.loadAsyncBulletinBoardData()
    }, 5000)

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
    async loadAsyncBulletinBoardData() {
      this.$axios
        .get(`/api/bulletinboard/`, {
          crossDomain: true
        })
        .then(({ data }) => {
          var oldMessages = this.kayanYaziWrapper.map(function(m) {
            return m.message
          })
          var oldMessagesString = oldMessages.join()

          var newMessages = data.data.map(function(m) {
            return m.message
          })
          var newMessagesString = newMessages.join()

          if (oldMessagesString !== newMessagesString) {
            this.kayanYaziWrapper = data.data
          }
        })
        .catch(error => {
          this.kayanYaziWrapper = []
          throw error
        })
    },
    secondsToFullHour(item) {
      if (!item) {
        return null
      }
      var date = new Date(null)
      date.setSeconds(item)
      const result = date.toISOString().substr(11, 5)
      return result
    },
    timestampToDateString(ts) {
      if (!ts) {
        return null
      }
      var dString = new Date(ts).toLocaleString('tr')
      return dString
    },
    async loadAsyncData() {
      console.log(this.$router.history.current)
      this.$axios
        .get(`/api/workorder?type=${this.$router.history.current.query.type}`, {
          crossDomain: true
        })
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
thead {
  background-color: #51586a;
}

.th-wrap {
  color: white;
  font-size: 14px;
  text-align: left;
}
.table-wrapper {
  background: #fafafa;
}
.table td,
.table th {
  padding: 0.25em 0.25em;
}
thead tr {
  height: 60px;
}
tbody tr {
  height: 60px;
}
tr th {
  height: 60px;
}
tr td {
  height: 60px;
}

tbody td span {
  height: 60px;
  overflow: hidden;
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

.perc-value {
  height: 1rem;
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

html {
  overflow-y: hidden;
}

body {
  background-color: #314150;
  height: 100%;
  margin: 0;
}

#__nuxt {
  height: 100vh;
}
#__layout {
  height: 100vh;
}
#main-content {
  height: 100vh;
  padding-top: 70px;
  padding-bottom: 50px;
}

#content {
  height: calc(100vh - (70px + 50px));
}
#header {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 999;
  width: 100%;
  height: 70px;
  background-color: rgb(44, 58, 71);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}
#footer {
  position: fixed;
  bottom: 0;
  left: 0;
  z-index: 999;
  width: 100%;
  height: 50px;
  background-color: rgb(44, 58, 71);
}
.marquee {
  font-family: 'Comic Sans MS';
  font-size: 1.5em;
  font-weight: bold;
  line-height: 1em;
  color: #ffffff;
  padding: 0.5em;
}

.table_label {
  height: 33px;
  margin-top: 5px !important;
  margin-bottom: 5px !important;
}
.b-table {
  height: calc(100% - 60px);
}
.table-wrapper {
  height: calc(100% - 40px);
}
table {
  height: 100%;
  width: 100%;
  table-layout: fixed;
}
</style>
