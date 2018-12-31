<template>
  <section class="section">
    <b-table
      :data="data"
      :loading="loading"

      :total="total"
      :per-page="perPage"
      :default-sort-direction="defaultSortOrder"
      :default-sort="[sortField, sortOrder]"
      paginated

      backend-pagination
      backend-sorting
      @page-change="onPageChange"
      @sort="onSort">

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
          numeric
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
          sortable
          centered>
          {{ new Date(props.row.statusDateTimeStamp).toLocaleDateString('tr') }}
        </b-table-column>

        <b-table-column
          field="workType"
          label="Is Kaynagi"
          width="500">
          {{ props.row.workType }}
        </b-table-column>
        <b-table-column
          field="estimatedDuration"
          label="Std Süre"
          width="500">
          {{ props.row.estimatedDuration }}
        </b-table-column>
        <b-table-column
          field="percentage"
          label="Durum"
          width="500">
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
  async asyncData({ $axios, params }) {
    const { data } = await $axios.get(`http://localhost:3001/${params.id}`)
    return { data }
  },
  name: 'HomePage',
  data() {
    return {
      total: 0,
      loading: false,
      sortField: 'workorderId',
      sortOrder: 'desc',
      defaultSortOrder: 'desc',
      page: 1,
      perPage: 10
    }
  },
  methods: {
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
