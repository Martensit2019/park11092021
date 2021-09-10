<template>
  <div>
    <data-table-wrapper
      :loaded="isLoading"
      :title="title"
      :tabsOpt="tabsOpt"
      :rowData="in_users"
      :columns="headerFields"
      @filtering="handleFiltering"
    >
      <template>
        <DataTable
          :header-fields="filteredHeaderFields"
          :data="in_users"
          :is-loading="!isLoading"
        >
          <template v-slot:role="props">
            <span>{{ props.rowData.role.name }}</span>
          </template>
          <template v-slot:cn_region="props">
            <span>{{ props.rowData.cn_region_name }}</span>
          </template>

          <template v-slot:cn_region="props">
            <span>{{ props.rowData.cn_region_name }}</span>
          </template>

          <template v-slot:active="props">
            <!-- <span>{{ props.rowData.active }}</span> -->
            <span class="checkbox d-flex justify-content-center mb-0">
              <input
                type="checkbox"
                :checked="props.rowData.active"
                :id="props.rowData.id"
                disabled
              />
              <label :for="props.rowData.id"></label>
            </span>
          </template>

          <Pagination
            slot="pagination"
            :page="params.currentPage"
            :total-items="totalItems"
            :items-per-page="params.perPage"
            @on-update="changePage"
            @update-current-page="updateCurrentPage"
          />

          <div class="items-per-page" slot="ItemsPerPage">
            <label>{{ $t("itemsPerPage") }}: </label>
            <ItemsPerPageDropdown
              :items-per-page="params.perPage"
              @on-update="updateItemsPerPage"
            />
          </div>
          <Spinner slot="spinner" />
        </DataTable>
      </template>
    </data-table-wrapper>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
import DataTableWrapper from "@/components/DataTableWrapper";
import Spinner from "@/components/vueSpinner";
import {
  DataTable,
  ItemsPerPageDropdown,
  Pagination
} from "@/components/vue-datatable";

export default {
  components: {
    DataTableWrapper,
    DataTable,
    ItemsPerPageDropdown,
    Pagination,
    Spinner
  },
  computed: {
    ...mapGetters({
      in_users: "store/in_users",
      currentPage: "store/page",
      itemsPerPage: "store/perPage",
      totalItems: "store/totalCount",
      isLoading: "store/loaded"
    }),
    filteredHeaderFields() {
      return this.headerFields.filter(item => item.hidden !== true);
    }
  },
  head() {
    return {
      title: this.$t(this.title)
    };
  },
  data() {
    return {
      title: "users",
      countries: [],
      regions: [],
      tabsOpt: {
        tabsOn: true,
        activeTab: "Tab1",
        mainTabTitle: "Внутренние пользователи"
      },
      headerFields: [
        {
          name: "full_name",
          label: "Ф.И.О.",
          sortable: false,
          editType: "input",
          rules: "required",
          paramByFiltering: "fio",
          paramByFilteringType: "string"
        },
        {
          name: "organization_name",
          label: "Название организации",
          sortable: false,
          editType: "none",
          hidden: true,
          view: true,
          filterable: false
        },
        {
          name: "current_role",
          altName: "role",
          label: "Должность",
          customElement: "role",
          sortable: false,
          editType: "select",
          dataType: { type: "object", label: "name" },
          paramByFiltering: "role"
        },
        {
          name: "cn_region_id",
          label: "Регион ТС",
          customElement: "cn_region",
          sortable: false,
          editType: "select",
          paramByFiltering: "region"
        },
        {
          name: "email",
          label: "Электронная почта",
          sortable: false,
          editType: "input",
          rules: "required|email",
          filterable: false
        },
        {
          name: "phone",
          label: "Телефон",
          sortable: false,
          editType: "input_phone",
          rules: "required",
          filterable: false
        },
        {
          name: "region_id",
          label: "Регион",
          sortable: false,
          editType: "select",          
          hidden: true,
          view: true,
          filterable: false
        },
        {
          name: "city_name",
          label: "Город",
          sortable: false,
          editType: "input",
          hidden: true,
          view: true,
          rules: "required",
          filterable: false
        },
        {
          name: "hired_at",
          label: "Дата приема",
          sortable: false,
          editType: "none",
          hidden: true,
          view: true,
          filterable: false
        },
        {
          name: "fired_at",
          label: "Дата увольнения",
          sortable: false,
          editType: "none",
          hidden: true,
          view: true,
          filterable: false
        },
        {
          name: "code_1c",
          label: "Код в 1С",
          sortable: false,
          editType: "input",
          view: false,
          filterable: false
        },
        {
          name: "last_used_at",
          label: "Дата авторизации",
          sortable: false,
          editType: "none",
          view: false,
          filterable: false
        },
        {
          name: "active",
          label: "Активен",
          customElement: "active",
          sortable: false,
          editType: "checkbox",
          view: true,
          paramByFiltering: "active"
        }
      ],

      params: {
        perPage: 6,
        currentPage: 1
      }
    };
  },
  mounted() {
    this.$nextTick(function() {
      this.fetch();
    });
  },
  // beforeRouteUpdate(to, from, next) {
  //   if (to.meta?.name.includes(this.title)) {
  //     this.fetch();
  //   }
  //   // console.log(to.meta.path && to.meta.path === this.$route.meta.path);
  //   next();
  // },
  watch: {
    params: {
      handler() {
        this.fetch();
      },
      deep: true
    },
    $route(to, from) {
      if (to.name === "admin-users") {
        this.fetch();
      }
    }
  },
  methods: {
    ...mapActions({
      loadInUsers: "store/loadInUsers",
      loadCnRegions: "store/loadCnRegions",
      loadRegions: "store/loadRegions"
    }),
    fetch: function() {
      this.$store.dispatch("store/loadInUsers", this.params);
      this.$store.dispatch("store/loadCnRegions");
      this.$store.dispatch("store/loadRegions");
    },
    updateItemsPerPage: function(itemsPerPage) {
      this.params.perPage = itemsPerPage;
    },
    changePage: function(currentPage) {
      this.params.currentPage = currentPage;
    },
    updateCurrentPage: function(currentPage) {
      this.params.currentPage = currentPage;
    },
    handleFiltering(filter) {
      this.params.currentPage = 1;
      if (Object.keys(filter).length) {
        const params = [this.params];
        this.params = Object.assign(...params, filter);
        this.tabsOpt.activeTab = "Tab1";
      } else {
        this.params = { perPage: 6, currentPage: 1 };
      }
      this.$store.dispatch("store/loadInUsers", this.params);
    }
  }
};
</script>
