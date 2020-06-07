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
                <div style="margin-bottom: 5px;">
                    <input type="text" v-on:input="onQuickFilterChanged()" id="quickFilter" placeholder="quick filter...">
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
  
    export default {
        name: 'App',
        data() {
            return {
      gridOptions: null,
      gridApi: null,
      columnApi: null,
      defaultColDef: null,
      columnDefs: null,
      sideBar: null,
      statusBar: null,
      modules: AllModules,
      enableRangeSelection: true,
      rowData: null,
              paginationPageSize: 10,
              paginationNumberFormatter: null
            }
        },
        components: {
            AgGridVue
        },
  beforeMount() {
    this.gridOptions = {};
    this.defaultColDef = {
      enableRowGroup: true,
      enablePivot: true,
      enableValue: true,
      width: 140,
      sortable: true,
      resizable: true,
      filter: true,
    };
    this.columnDefs = [
    {
        headerName: 'Shift Details',
        children: [
            { headerName: 'Site Name', field: 'athlete' },
            { headerName: 'Type', field: 'age' },
            { headerName: 'Officer', field: 'year' },
            { headerName: 'Date', field: 'date' },
            { headerName: 'Day', field: 'day' }
        ]
    },
    {
        headerName: 'Staff Details',
        children: [
            { headerName: 'Start', field: 'start' },
            { headerName: 'End', field: 'end' },
            { headerName: 'HR', field: 'hr' },
            { headerName: 'Break', field: 'break' },
            { headerName: 'Pay Rate', field: 'payrate'},
            { headerName: 'Amount', field: 'amount', hide: true}
        ]
    },
    {
        headerName: 'Site Details',
        children: [
            { headerName: 'Site Start', field: 'start', hide: true },
            { headerName: 'Site End', field: 'end', hide: true },
            { headerName: 'Site HR', field: 'hr', hide: true },
            { headerName: 'Change Rate', field: 'changeRate', hide: true },
            { headerName: 'Change Amount', field: 'changeAmount', hide: true},
            { headerName: 'Expense', field: 'expense', hide: true },
             { headerName: 'Status', field: 'status', hide: true }
             
        ]
    } 
    ];
    this.sideBar = {
      toolPanels: [
        'filters',
        {
          id: 'columns',
          labelDefault: 'Columns',
          labelKey: 'columns',
          iconKey: 'columns',
          toolPanel: 'agColumnsToolPanel',
          toolPanelParams: { suppressSyncLayoutWithGrid: true },
        },
      ],
    };
    this.statusBar = {
      statusPanels: [
        {
          statusPanel: 'agTotalRowCountComponent',
          align: 'left',
          key: 'totalRowComponent',
        },
        {
          statusPanel: 'agFilteredRowCountComponent',
          align: 'left',
        },
        {
          statusPanel: 'agSelectedRowCountComponent',
          align: 'center',
        },
        {
          statusPanel: 'agAggregationComponent',
          align: 'right',
        },
      ],
    };
    
  },
  mounted() {
    this.gridApi = this.gridOptions.api;
    this.gridColumnApi = this.gridOptions.columnApi;
  },
   methods: {
         onQuickFilterChanged() {
      this.gridApi.setQuickFilter(document.getElementById('quickFilter').value);
    },
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
        'https://raw.githubusercontent.com/ag-grid/ag-grid/master/grid-packages/ag-grid-docs/src/olympicWinnersSmall.json'
      );
      httpRequest.send();
      httpRequest.onreadystatechange = () => {
        if (httpRequest.readyState === 4 && httpRequest.status === 200) {
          updateData(JSON.parse(httpRequest.responseText));
        }
      };
  }}
    }
</script>
<style lang="scss">
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-alpine.css";
</style>
