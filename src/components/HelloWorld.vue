<template>
  <div id="app">
    <div style="height: calc(100vh - 100px)">
                <div class="example-header">
                    Page Size:
                    <select v-on:change="onPageSizeChanged()" id="page-size">
                        <option value="10" selected="">10</option>
                        <option value="100">100</option>
                        <option value="500">500</option>
                        <option value="1000">1000</option>
                    </select>
                </div>
            <ag-grid-vue
                style="width: 100%; height: 100%;"
                class="ag-theme-alpine"
                id="myGrid"
                :gridOptions="gridOptions"
                @grid-ready="onGridReady"
                :columnDefs="columnDefs"
                :defaultColDef="defaultColDef"
                :modules="modules"
                :pagination="true"
                :paginationPageSize="paginationPageSize"
                :sideBar="sideBar"
                :rowData="rowData"></ag-grid-vue>
    </div>
  </div>
</template>

<script>
    import { AgGridVue } from '@ag-grid-community/vue';
    import { AllModules } from '@ag-grid-enterprise/all-modules';
    import { ClientSideRowModelModule } from '@ag-grid-community/client-side-row-model';
    import { RowGroupingModule } from '@ag-grid-enterprise/row-grouping';
    import { MenuModule } from '@ag-grid-enterprise/menu';
    import { SetFilterModule } from '@ag-grid-enterprise/set-filter';
    import { ColumnsToolPanelModule } from '@ag-grid-enterprise/column-tool-panel';

    export default {
        name: 'App',
        data() {
            return {
              gridOptions: null,
              gridApi: null,
              columnApi: null,
              columnDefs: null,
              defaultColDef: null,
              defaultColGroupDef: null,
              columnTypes: null,
                    modules: [
        ClientSideRowModelModule,
        RowGroupingModule,
        MenuModule,
        SetFilterModule,
        ColumnsToolPanelModule,
        AllModules
      ],
              rowData: null,
              sideBar: null,
              paginationPageSize: 10,
              paginationNumberFormatter: null
            }
        },
        components: {
            AgGridVue
        },
  beforeMount() {
    this.gridOptions = {};
    this.columnDefs = [
      { field: 'athlete' },
      {
        field: 'age',
        filter: 'agNumberColumnFilter',
        maxWidth: 100,
      },
      { field: 'country' },
      {
        field: 'year',
        maxWidth: 100,
      },
      {
        field: 'date',
        filter: 'agDateColumnFilter',
        filterParams: {
          comparator: (filterLocalDateAtMidnight, cellValue) => {
            var dateAsString = cellValue;
            if (dateAsString == null) return -1;
            var dateParts = dateAsString.split('/');
            var cellDate = new Date(
              Number(dateParts[2]),
              Number(dateParts[1]) - 1,
              Number(dateParts[0])
            );
            if (filterLocalDateAtMidnight.getTime() == cellDate.getTime()) {
              return 0;
            }
            if (cellDate < filterLocalDateAtMidnight) {
              return -1;
            }
            if (cellDate > filterLocalDateAtMidnight) {
              return 1;
            }
          },
          browserDatePicker: true,
        },
      },
      { field: 'sport' },
      {
        field: 'gold',
        filter: 'agNumberColumnFilter',
      },
      {
        field: 'silver',
        filter: 'agNumberColumnFilter',
      },
      {
        field: 'bronze',
        filter: 'agNumberColumnFilter',
      },
      {
        field: 'total',
        filter: false,
      },
    ];
    this.defaultColDef = {
      flex: 1,
      minWidth: 100,
      editable: true,
      filter: 'agTextColumnFilter',
      floatingFilter: true,
      resizable: true,
      enablePivot: true,
    };
    this.sideBar = ['column', 'filter'];
  },
  mounted() {
    this.gridApi = this.gridOptions.api;
    this.gridColumnApi = this.gridOptions.columnApi;
  },
   methods: {
    onPageSizeChanged() {
      var value = document.getElementById('page-size').value;
      this.gridApi.paginationSetPageSize(Number(value));
    },
    onGridReady(params) {
      const httpRequest = new XMLHttpRequest();
      const updateData = data => {
        this.rowData = data;
        params.api.paginationGoToPage(4);
      };

      httpRequest.open(
        'GET',
        'https://raw.githubusercontent.com/ag-grid/ag-grid/master/grid-packages/ag-grid-docs/src/olympicWinners.json'
      );
      httpRequest.send();
      httpRequest.onreadystatechange = () => {
        if (httpRequest.readyState === 4 && httpRequest.status === 200) {
          updateData(JSON.parse(httpRequest.responseText));
        }
      };
    },
  },
    }
</script>
<style lang="scss">
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-alpine.css";
</style>
