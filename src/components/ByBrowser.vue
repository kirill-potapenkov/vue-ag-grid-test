<template>
  <div class='w-full' style='height: 48vh'>
    <div class="p-2 m-2">
      <span v-if="isFetching">Fetching data...</span>
      <span v-if="isProcessing">Processing data...</span>
      <label v-if="!isFetching && !isProcessing" for="rowNumSelector">
        Remote row number:
        <select v-model="pageSize" id="rowNumSelector">
          <option>1000</option>
          <option>3000</option>
          <option>6000</option>
          <option>7000</option>
          <option>9000</option>
          <option>10000</option>
          <option>50000</option>
          <option>100000</option>
          <option>150000</option>
          <option>200000</option>
        </select>
      </label>
      <secondary-button @click="onBtNormal()">
        1 - Grouping Active
      </secondary-button>
      <secondary-button @click="onBtPivotMode()">
        2 - Grouping Active with Pivot Mode
      </secondary-button>
      <secondary-button @click="onBtFullPivot()">
        3 - Grouping Active with Pivot Mode and Pivot Active
      </secondary-button>

    </div>

    <ag-grid-vue
        style='width: 100%; height: 100%;'
        :class='themeClass'
        :columnDefs='columnDefs'
        @grid-ready='onGridReady'
        @row-data-updated='onRowDataUpdated'
        :defaultColDef='defaultColDef'
        :autoGroupColumnDef='autoGroupColumnDef'
        :rowData='rowData'
        :loading='true'
        :sideBar="sideBar"
        :groupDisplayType="groupDisplayType"

    ></ag-grid-vue>
  </div>
</template>

<script>
import 'ag-grid-enterprise';
import {onBeforeMount, ref, shallowRef, watch} from 'vue';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-quartz.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';
import {AgGridVue} from 'ag-grid-vue3';
import {ClientSideRowModelModule} from '@ag-grid-community/client-side-row-model';
import {ModuleRegistry} from '@ag-grid-community/core';
import {RowGroupingModule} from '@ag-grid-enterprise/row-grouping';
import axios from "axios";
import SecondaryButton from "@/components/SecondaryButton.vue";

ModuleRegistry.registerModules([ClientSideRowModelModule, RowGroupingModule]);


export default {
  name: 'ByBrowser',
  components: {
    SecondaryButton,
    AgGridVue,
  },
  setup() {
    const columnDefs = ref([
      {field: "country", pivot: true, enablePivot: true, aggFunc: "count"},
      {field: "city", pivot: true, enablePivot: true, aggFunc: "count"},
      {field: "event_ts",},
      {field: "enginename",},
      {field: "region",},
      {field: "browsername", rowGroup: true, enableRowGroup: true, enablePivot: true},
      {field: "browserversion", rowGroup: true, enableRowGroup: true, enablePivot: true},
      {field: "connection",},
      {field: "host", aggFunc: "count"},
      {field: "isbot", pivot: true, enablePivot: true, aggFunc: "count"},
      {field: "osname",},
      {field: "osversion",}
    ]);

    const groupDisplayType = ref('multipleColumns');

    const gridApi = shallowRef();
    const defaultColDef = ref({
      flex: 1,
      minWidth: 150,
    });

    const autoGroupColumnDef = ref({
      headerValueGetter: params => `${params.colDef.headerName} Group Column`,
      minWidth: 220,
      cellRendererParams: {
        suppressCount: false,
      }
    });
    const sideBar = ref(null);
    const rowData = ref(null);

    onBeforeMount(() => {
      autoGroupColumnDef.value = {
        minWidth: 250,
      };
      sideBar.value = "columns";
    });

    const isFetching = ref(true);
    const isProcessing = ref(false);

    const pageSize = ref(10000);
    const page = 1;
    const token = 'p.eyJ1IjogIjQ5ODM1ZDNlLTFlNzktNDA1Yi1hMWUxLWNhNjZmNzk2MzMyMCIsICJpZCI6ICI5NGI4NWI1NC1mNDViLTRmMWEtYjM0Ni05M2ViNThiOWZjZjYiLCAiaG9zdCI6ICJldV9zaGFyZWQifQ.-i23srOyg9PHfo6WsSw2eVhZmcDn_fhdFYCCclPC4nQ';

    const updateData = (data) => (rowData.value = data);
    const fetchData = () => {
      isFetching.value = true;
      axios
          .get(`https://api.tinybird.co/v0/pipes/logAnalytincs.json?page_size=${pageSize.value}&page=${page}&token=${token}`)
          .then(data => {
            isFetching.value = false;
            isProcessing.value = true;
            updateData(data.data.data)
          })
    };

    const onGridReady = (params) => {
      gridApi.value = params.api;
      fetchData();
    };

    watch(pageSize, async () => {
      gridApi.value.setGridOption('loading', true);
      fetchData();
    });

    const onRowDataUpdated = () => {
      console.log('Row Data Updated');
      isProcessing.value = false;
      gridApi.value.setGridOption('loading', false);
      gridApi.value.setGridOption('suppressAutoSize', true);
    };

    const onBtNormal = () => {
      gridApi.value.setGridOption("pivotMode", false);
      gridApi.value.applyColumnState({
        state: [
          {colId: "browsername", rowGroup: true},
          {colId: "browserversion", rowGroup: true},
        ],
        defaultState: {
          pivot: false,
          rowGroup: false,
        },
      });
    };
    const onBtPivotMode = () => {
      gridApi.value.setGridOption("pivotMode", true);
      gridApi.value.applyColumnState({
        state: [
          {colId: "browsername", rowGroup: true},
          {colId: "browserversion", rowGroup: true},
        ],
        defaultState: {
          pivot: false,
          rowGroup: false,
        },
      });
    };
    const onBtFullPivot = () => {
      gridApi.value.setGridOption("pivotMode", true);
      gridApi.value.applyColumnState({
        state: [
          {colId: "browsername", rowGroup: true},
          {colId: "browserversion", pivot: true},
        ],
        defaultState: {
          pivot: false,
          rowGroup: false,
        },
      });
    };

    return {
      columnDefs,
      gridApi,
      defaultColDef,
      autoGroupColumnDef,
      rowData,
      onGridReady,
      onRowDataUpdated,
      themeClass: 'ag-theme-quartz',
      pageSize,
      isFetching,
      isProcessing,
      groupDisplayType,
      sideBar,
      onBtNormal,
      onBtPivotMode,
      onBtFullPivot
    };
  },

};
</script>
