<template>
  <div>
    <div class="top">
      <h1 class="text-uppercase">
        <span v-if="$route.meta.name === 'create'">{{ $t("create") }}</span>
        <span v-else>{{ $t("edit") }}</span>
      </h1>
    </div>
    <ValidationObserver ref="edit_observer" v-slot="{ invalid, handleSubmit }">
      <div
        class="projectService"
        v-for="(column, idx) in columns"
        :key="column.name + '_' + idx"
        v-show="column.editType !== 'none' && column"
      >
        <div
          class="projectService-label long"
          v-if="column.editType !== 'none'"
        >
          <!-- {{column}} -->
          {{ column.label }}:
        </div>

        <div class="projectService-content">
          <!-- {{rowData}} -->
          <ValidationProvider
            :rules="column.rules ? column.rules : ''"
            v-slot="{ errors, classes }"
            mode="passive"
          >
            <div
              :class="classes"
              v-if="column.editType === 'input' && column.altName"
            >
              <input
                class="long"
                type="text"
                v-model="rowData[column.altName]"
                :class="{ error: errors.length }"
                :required="column.req === true"
              />
            </div>

            <div
              :class="classes"
              v-if="column.editType === 'input' && !column.altName"
            >
              <input
                class="long"
                type="text"
                v-model="rowData[column.name]"
                :required="column.req === true"
                :class="{ error: errors.length }"
              />
            </div>

            <div :class="classes" v-if="column.editType === 'input_number'">
              <input
                class="long"
                type="number"
                min="0"
                v-model.number="rowData[column.name]"
              />
            </div>
            <!-- {{ column.name }} <br /><br /> -->
            <!-- {{ rowData }} -->
            <div :class="classes" v-if="column.editType === 'input_phone'">
              <input
                class="long"
                type="text"
                :class="{ error: errors.length }"
                name="phone"
                v-model.lazy="rowData[column.name]"
                v-imask="masks[1]"
                :placeholder="getPhonePlaceholder(1)"
                :required="column.req === true"
              />
            </div>

            <div :class="classes" v-if="column.editType === 'input_number'">
              <number-field class="long" v-model="rowData[column.name]" />
            </div>

            <template v-if="column.editType === 'select' && column.altName">
              <v-select
                v-if="dicts[column.altName]"
                class="long"
                :class="[classes, column.multiple ? 'multiple' : '']"
                :options="dicts[column.altName]"
                :multiple="column.multiple"
                label="name"
                :reduce="item => item.id"
                v-model="rowData[column.altName]"
              >
                <template #no-options="{ search, searching, loading }">
                  {{ $t("notFoundMsg") }}
                </template>
              </v-select>
            </template>

            <template v-if="column.editType === 'select' && !column.altName">
              <v-select
                v-if="dicts[column.name]"
                class="long"
                :class="[classes, column.multiple ? 'multiple' : '']"
                :options="dicts[column.name]"
                :multiple="column.multiple"
                label="name"
                :reduce="item => item.id"
                v-model="rowData[column.name]"
              >
                <template #no-options="{ search, searching, loading }">
                  {{ $t("notFoundMsg") }}
                </template>
              </v-select>
            </template>

            <div :class="classes" v-if="column.editType === 'upload'">
              <input
                class="custom-upload"
                type="file"
                accept=".jpg, .jpeg, .png"
                :required="column.req === true"
              />
            </div>

            <div class="checkbox mb-0" v-if="column.editType === 'checkbox'">
              <input
                type="checkbox"
                v-model="rowData[column.name]"
                :id="column.name + '_' + idx"
              />
              <label :for="column.name + '_' + idx"></label>
            </div>

            <span class="error">{{ errors[0] }}</span>
          </ValidationProvider>
        </div>
      </div>

      <div v-if="errors.length" class="api-errors">
        <div v-for="(item, key) in errors" :key="key" class="api-error">
          {{ item.label }}: {{ item.errors }}
        </div>
      </div>

      <div class="group-btns" v-if="buttons">
        <div class="btn" @click="handleSubmit(action)">
          <!-- <div class="btn" @click="action()" :class="{ disabled: invalid }"> -->
          {{ $t("save") }}
        </div>
        <div @click="back()" class="btn">{{ $t("back") }}</div>
        <!-- <nuxt-link :to="`${$route.path}`" class="btn">{{ $t("back") }}</nuxt-link> -->
      </div>
    </ValidationObserver>
    <div class="img-preview" v-if="rowData && rowData.img_url">
      <img :src="rowData.img_url" alt="" />
    </div>
  </div>
</template>

<script>
import auth from "@/mixins/auth";
import { loadCountries, loadRegions } from "~/src/queries";
import { ValidationObserver, ValidationProvider } from "vee-validate";
import { mapGetters, mapActions } from "vuex";
import NumberField from "./NumberField";
import error from "../layouts/error";
import loadData from "./loadData";

export default {
  layout: "auth",
  mixins: [auth, loadData],
  name: "editPage",
  data() {
    return {
      rowData: {},
      countries: [],
      regions: [],
      errors: [],
      infiniteScroll: {
        observer: null,
        limit: 5
      },
      // нужно для исключения полей которые не менялись
      currentData: {
        email: null,
        phone: null
      }
    };
  },
  props: {
    columns: { default: () => [], type: Array },
    buttons: { default: false, type: Boolean }
  },
  components: {
    NumberField,
    ValidationObserver,
    ValidationProvider
  },
  computed: {
    filteredCols() {
      return this.columns.reduce((arr, key) => {
        if (key.editType !== "none") {
          if (key.altName) {
            arr.push(key.altName);
          } else if (key.name) {
            arr.push(key.name);
          }
        }
        return arr;
      }, []);
    },
    dicts() {
      return {
        director_id: this.$store.state.store.directors,
        request_type: this.$store.state.store.requestTypes,
        region_id: this.$store.state.store.regions,
        country_id: this.$store.state.store.countries,
        coordinator_id: this.$store.state.store.coordinators,
        responsible_tp_user_id: this.$store.state.store.responsibleTp,
        responsible_lo_user_id: this.$store.state.store.responsibleLo,
        responsible_pi_user_id: this.$store.state.store.responsiblePi,
        cn_regions: this.$store.state.store.regions,
        cn_region_id: this.$store.state.store.cn_regions,
        role: this.$store.state.roles.roles.slice(0, 9),
        macroregions: this.$store.state.store.macroregions
      };
    }
  },
  async mounted() {
    if (this.$route.meta.name === "edit") {
      this.fetch();
    } else if (this.$route.meta.name === "create") {
      this.rowData = this.filteredCols.reduce(
        (obj, key) => ({
          ...obj,
          [key]: null
        }),
        {}
      );
    }
  },
  created() {
    this.loadData(this.columns);
  },
  methods: {
    custom() {
      return { on: ["search:blur", "input", "change", "blur"] };
    },
    async fetch() {
      await this.$axios
        .get(`api${this.$route.meta.path}/${this.$route.params.id}`)
        .then(response => {
          let res = response.data.data || response.data;
          for (const key in this.currentData) {
            if (res[key]) {
              this.currentData[key] = res[key];
            }
          }
          this.rowData = this.filteredCols.reduce((obj, key, index) => {
            const filedIndex = this.columns.findIndex(
              item => item.altName === key || item.name === key
            );
            if (
              this.columns[filedIndex].dataType &&
              this.columns[filedIndex].dataType.type === "object"
            ) {
              return { ...obj, [key]: res[this.columns[filedIndex].name].id };
            } else {
              if (res[key].constructor.name === "Array") {
                return { ...obj, [key]: res[key].map(item => item.id) };
              }
              return { ...obj, [key]: res[key] };
            }
          }, {});
        });
    },
    open() {
      this.$store.dispatch("store/loadCnRegions", { currentPage: 2 });
    },
    onOpen(fetch_name) {
      if (this.$refs[fetch_name] && this.$refs[fetch_name].length) {
        const loading = this.$refs[fetch_name][0].toggleLoading;
        loading(true);
        if (loading) {
          this.getData(fetch_name, loading);
        }
      }
    },
    async getData(fetch_name, loading) {
      let api_url = "";
      if (fetch_name === "countries") {
        api_url = "api/";
      } else {
        api_url = "api/admin/";
      }
      await this.$axios.get(`${api_url}${fetch_name}`).then(response => {
        const res = response.data || response;
        let obj = Object.assign(...this.options, {
          [fetch_name]: ([fetch_name] = response.data.data || response.data)
        });
        this.options = [obj];
        // this.$refs[fetch_name][0].totalItems = res.meta
        //   ? res.meta.total
        //   : this.optionss[fetch_name].length;
        // this.limit =
        //   this.optionss[fetch_name].length < this.limit
        //     ? this.optionss[fetch_name].length
        //     : this.limit;
        // console.log(this.$refs[fetch_name][0]);
        loading(false);
      });
    },
    async action() {
      let valid = null;
      let requestData = {};

      //валидация полей
      await this.$refs.edit_observer.validate().then(result => {
        valid = result;
      });
      if (!valid) {
        return;
      }

      // исключить поля email и phone для user если не изменились
      for (const key in this.rowData) {
        if (this.currentData[key] !== this.rowData[key]) {
          requestData[key] = this.rowData[key];
        }
      }

      if (this.$route.meta.name === "create") {
        await this.$axios
          .post(`api${this.$route.meta.path}`, this.rowData)
          .then(response => {
            this.$router.push(this.$route.meta.path);
          })
          .catch(error => {
            if (error.response.status === 422) {
              //парсим ошибки со стороны сервера
              this.errors = this.parseError(error.response.data.errors);
            }
          });
      }
      if (this.$route.meta.name === "edit") {
        if (this.rowData.current_role) {
          // this.rowData.role = this.rowData.current_role.id;
          if (Object.keys(this.rowData.current_role).length) {
            this.rowData.role = this.rowData.current_role.id;
          } else {
            this.rowData.role = this.rowData.current_role;
          }
        }
        if (this.rowData.phone) {
          this.rowData.phone = this.rowData.phone.split(" ").join("");
        }

        await this.$axios
          .put(
            `api${this.$route.meta.path}/${this.$route.params.id}`,
            requestData
          )
          .then(response => {
            this.$router.push(
              `${this.$route.meta.path}/${this.$route.params.id}`
            );
          })
          .catch(error => {
            if (error.response.status === 422) {
              //парсим ошибки со стороны сервера
              this.errors = this.parseError(error.response.data.errors);
            }
          });
      }
    },
    parseError(errors) {
      const result = [];
      for (const key in errors) {
        const filedIndex = this.columns.findIndex(
          item => item.altName === key || item.name === key
        );
        result.push({
          label: null, // this.columns[filedIndex].label,
          errors: errors[key].join(" ")
        });
      }
      return result;
    },
    back() {
      if (this.$route.meta.name === "edit") {
        this.$router.push({
          path: `${this.$route.meta.path}/${this.$route.params.id}`   });
      } else {
        this.$router.push({ path: this.$route.meta.path });
      }
    }
  }
};
</script>
<style lang="sass">
.multiple
  height: auto !important
  .vs__dropdown-toggle
    height: auto !important
    min-height: 24px
  .vs__selected
    height: 22px
  button
    min-width: 10px
    border: none

.api-errors
  font-size: 12px
  color: red
  margin-bottom: 15px
</style>
