<template>
  <div>
    <div class="top">
      <h1 class="text-uppercase">{{ $t("view") }}</h1>
    </div>
    <!-- rowData {{rowData}}<br><br> -->
    <!-- dicts: {{columns}}<br><br> -->
    <!-- cn_regions: {{dicts.role_id}} -->
    <div
      class="projectService"
      v-for="(column, idx) in columns"
      :key="column.name + idx"
      v-show="column.view !== false"
    >
      <div class="projectService-label long" v-if="column.view !== false">
        <!-- {{ column }}<br /><br /> -->
        {{ column.label }}:
      </div>

      <div class="projectService-content">
        <div>
          <span
            v-if="
              dicts &&
                dicts[column.altName] &&
                !(column.dataType && column.dataType.type === 'array')
            ">
            {{
              dicts[column.altName]
                .filter(item => item.id === rowData[column.altName])
                .map(item => item.name)[0]
            }}
          </span>
          <span v-else>{{ viewData(rowData[column.altName], column) }}</span>

        </div>
        <div>
          <span
            v-if="
              dicts &&
                dicts[column.name] &&
                !(column.dataType && column.dataType.type === 'array')
            "
          >
            {{
              dicts[column.name]
                .filter(item => item.id === rowData[column.name])
                .map(item => item.name)[0]
            }}
          </span>
         <span v-else-if="column.editType === 'checkbox'"></span>
          <span v-else> {{ viewData(rowData[column.name], column) }}</span>
        </div>
         <div class="checkbox mb-0" v-if="column.editType === 'checkbox'">
            <input
              type="checkbox"
              v-model="rowData[column.name]"
              :id="column.name + '_' + idx"
            />
            <label :for="column.name + '_' + idx"></label>
          </div>
          <!-- <span v-else>{{ rowData[column.name] }}</span> -->
      </div>
    </div>

    <div class="group-btns" v-if="buttons">
      <nuxt-link :to="`${$route.params.id}/edit`" class="btn">{{
        $t("edit")
      }}</nuxt-link>
      <div class="btn" @click="deleteRow">{{ $t("delete") }}</div>
      <nuxt-link :to="$route.meta.path" class="btn">
        {{ $t("back") }}
      </nuxt-link>
    </div>
  </div>
</template>

<script>
import loadData from "./loadData";
import DataTableItemsDropdown from "./vue-datatable/components/ItemsPerPageDropdown";
import { loadCountries, loadRegions, register } from "~/src/queries";

export default {
  name: "viewPage",
  components: { DataTableItemsDropdown },
  mixins: [loadData],
  data() {
    return {
      rowData: {},
      regions: [],
      cn_regions: [],
      roles: []
    };
  },
  props: {
    columns: { default: () => [], type: Array },
    buttons: { default: false, type: Boolean }
  },
  computed: {
    dictsKeys() {
      let f = Object.keys(this.rowData).filter(key =>
        Object.keys(this.dicts).includes(key)
      );
      return Object.keys(this.dicts);
    },
    dicts() {
      return {
        director_id: this.$store.state.store.directors,
        request_type: this.$store.state.store.requestTypes,        
        country_id: this.$store.state.store.countries,
        region_id: this.$store.state.store.regions,
        coordinator_id: this.$store.state.store.coordinators,
        responsible_tp_user_id: this.$store.state.store.responsibleTp,
        responsible_lo_user_id: this.$store.state.store.responsibleLo,
        responsible_pi_user_id: this.$store.state.store.responsiblePi,
        cn_region_id: this.$store.state.store.cn_regions,
        role: this.$store.state.roles.roles
      };
    }
  },
  async mounted() {
    if (this.$route.params.id) {
      await this.fetch();
    }
  },
  created() {
    // loadCountries().then(countries => (this.countries = countries));
    // loadRegions(this.user.country_id).then(regions => (this.regions = regions));
    if (this.rowData && this.columns) {
      if (
        this.rowData.fired_at !== null &&
        this.$route.name === "admin-users-view"
      ) {
        // this.columns[9] = false;
      }
    }
    this.loadData(this.columns);
  },
  methods: {
    async fetch() {
      await this.$axios
        .get(`api${this.$route.meta.path}/${this.$route.params.id}`)
        .then(response => {
          const fields = response.data.data || response.data;
          
           if (fields.fired_at === null) {
            this.columns[9].view = false;
          }
          if (!fields.organization_name) {
            this.columns[1].view = false;
          }
          if (fields.cn_region_id === null) {
            this.columns[3].view = false;
          }
          const row = [];
          for (const key in this.columns) {
            if (
              this.columns[key].dataType &&
              this.columns[key].dataType.type === "object"
            ) {
              row.push(fields[this.columns[key].name].id);
            } else {
              row.push(fields[this.columns[key].name]);
            }
          }
          this.rowData = row.reduce((obj, current, index) => {
            return {
              ...obj,
              [this.columns[index].altName || this.columns[index].name]: current
            };
          }, {});
        });
    },
    async deleteRow() {
      const confirm = window.confirm("Удалить запись?");
      if (confirm) {
        await this.$axios
          .request(`api${this.$route.meta.path}/${this.$route.params.id}`, {
            method: "delete"
          })
          .then(response => {
            this.$router.push(this.$route.meta.path);
          });
      }
    },
    viewData(data, header) {
      if (!header.dataType || !data) {
        return data;
      }

      switch (header.dataType.type) {
        case "array":
          if (!data.length) {
            return null;
          }
          return data.reduce(
            (accum, value) => `${accum} ${value[header.dataType.label]}`,
            ""
          );
        case "object":
          return data[header.dataType.label];
        default:
          return data;
      }
    }
  }
};
</script>
