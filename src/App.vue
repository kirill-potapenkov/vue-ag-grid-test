<template>
  <div class='w-full' style='height: 48vh'>
    <div class="p-2 m-2">
      <span v-if="isFetching">Fetching data...</span>
      <span v-if="isProcessing">Processing data...</span>
      <label v-if="!isFetching && !isProcessing" for="rowNumSelector">
        Remote row number:
        <select v-model="pageSize" id="rowNumSelector">
          <option>1000</option>
          <option>2000</option>
          <option>3000</option>
          <option>4000</option>
          <option>5000</option>
          <option>6000</option>
          <option>7000</option>
          <option>8000</option>
          <option>9000</option>
          <option>10000</option>
          <option>100000</option>
          <option>200000</option>
        </select>
      </label>
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

ModuleRegistry.registerModules([ClientSideRowModelModule, RowGroupingModule]);


export default {
  name: 'App',
  components: {
    AgGridVue,
  },
  setup() {
    const columnDefs = ref([
      {
        field: 'acceptcharset',
      },
      {
        field: 'acceptencoding',
      },
      {
        field: 'acceptlanguage',
      },
      {
        field: 'browsername', rowGroup: true, hide: true
      },
      {
        field: 'browserversion', rowGroup: true, hide: true
      },
      {
        field: 'cachecontrol',
      },
      {
        field: 'city',
      },
      {
        field: 'connection',
      },
      {
        field: 'contentlength',
      },
      {
        field: 'contenttype',
      },
      {
        field: 'country',
      },
      {
        field: 'cpuarchitecture',
      },
      {
        field: 'devicemodel',
      },
      {
        field: 'devicetype',
      },
      {
        field: 'devicevendor',
      },
      {
        field: 'enginename',
      },
      {
        field: 'engineversion',
      },
      {
        field: 'event_ts',
      },
      {
        field: 'from',
      },
      {
        field: 'headers',
      },
      {
        field: 'host',
      },
      {
        field: 'ip_address',
      },
      {
        field: 'isbot',
      },
      {
        field: 'latitude',
      },
      {
        field: 'log_level',
      },
      {
        field: 'log_message',
      },
      {
        field: 'longitude',
      },
      {
        field: 'method',
      },
      {
        field: 'origin',
      },
      {
        field: 'osname',
      },
      {
        field: 'osversion',
      },
      {
        field: 'protocol',
      },
      {
        field: 'referer',
      },
      {
        field: 'region',
      },
      {
        field: 'url',
      },
      {
        field: 'useragent',
      },
      {
        field: 'via',
      },
      {
        field: 'xforwaredforip',
      }

    ]);
    const gridApi = shallowRef();
    const defaultColDef = ref({
      flex: 1,
      minWidth: 100,
    });

    const autoGroupColumnDef = ref(null);
    const rowData = ref(null);

    onBeforeMount(() => {
      autoGroupColumnDef.value = {
        minWidth: 200,
      };
    });

    const isFetching = ref(true);
    const isProcessing = ref(false);

    const pageSize = ref(2000);
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
    };
  },

};
</script>
