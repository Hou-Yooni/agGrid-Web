<template>
  <ag-grid-vue
    style="width: 100%; height: 500px; text-align: left"
    class="flex-grow-1 flex-shrink-1 ag-theme-alpine-dark"
    :gridOptions="gridOptions"
    :columnDefs="columnDefs"
    :rowData="rowData"
    :sideBar="sideBar"
    :modules="modules"
    :defaultColDef="{
      sortable: true,
      resizable: true,
      filter: true
    }"
    :groupHeaders="true"
    :suppressRowClickSelection="true"
    rowSelection="multiple"
    @grid-ready="onGridReady"
    @model-updated="onModelUpdated"
    @row-selected="onRowSelected"
  >
  </ag-grid-vue>
</template>

<script>
import { AgGridVue } from "@ag-grid-community/vue";
import { AllModules } from "@ag-grid-enterprise/all-modules";
import { ProficiencyFilter } from "./proficiencyFilter";
import { SkillFilter } from "./skillFilter";
import DateComponent from "./DateComponent.vue";
import HeaderGroupComponent from "./HeaderGroupComponent.vue";
import RefData from "./refData";

export default {
  data() {
    return {
      gridOptions: null,
      api: null,
      columnDefs: null,
      rowData: null,
      sideBar: false,
      rowCount: null,
      modules: AllModules
    };
  },
  components: {
    AgGridVue,
    SkillFilter
  },
  methods: {
    createRowData() {
      const rowData = [];

      for (let i = 0; i < 200; i++) {
        const countryData = RefData.COUNTRIES[i % RefData.COUNTRIES.length];
        rowData.push({
          name:
            RefData.FIRST_NAMES[i % RefData.FIRST_NAMES.length] +
            " " +
            RefData.LAST_NAMES[i % RefData.LAST_NAMES.length],
          skills: {
            android: Math.random() < 0.4,
            html5: Math.random() < 0.4,
            mac: Math.random() < 0.4,
            windows: Math.random() < 0.4,
            css: Math.random() < 0.4
          },
          dob: RefData.DOBs[i % RefData.DOBs.length],
          address: RefData.ADDRESSES[i % RefData.ADDRESSES.length],
          years: Math.round(Math.random() * 100),
          proficiency: Math.round(Math.random() * 100),
          country: countryData.country,
          continent: countryData.continent,
          language: countryData.language,
          mobile: createRandomPhoneNumber(),
          landline: createRandomPhoneNumber()
        });
      }

      this.rowData = rowData;
    },
    createColumnDefs() {
      this.columnDefs = [
        {
          headerName: "#",
          minWidth: 60,
          width: 60,
          checkboxSelection: true,
          sortable: false,
          suppressMenu: true,
          pinned: true
        },
        {
          headerName: "Employee",
          headerGroupComponentFramework: HeaderGroupComponent,
          children: [
            {
              headerName: "Name",
              field: "name",
              width: 150,
              pinned: true
            },
            {
              headerName: "Country",
              field: "country",
              width: 150,
              cellRenderer: countryCellRenderer,
              pinned: true,
              filterParams: {
                cellRenderer: countryCellRenderer,
                cellHeight: 20
              },
            },
            {
              headerName: "DOB",
              field: "dob",
              width: 120,
              pinned: true,
              cellRenderer: params => {
                return (
                  params.value.getFullYear() +
                  "/" +
                  this.pad(params.value.getDate(), 2) +
                  "/" +
                  this.pad(params.value.getMonth() + 1, 2)
                );
              },
              filter: "agDateColumnFilter",
              columnGroupShow: "open"
            }
          ]
        },
        {
          headerName: "IT Skills",
          children: [
            {
              headerName: "Skills",
              width: 125,
              sortable: false,
              cellRenderer: skillsCellRenderer,
              filter: SkillFilter
            },
            {
              headerName: "Proficiency",
              field: "proficiency",
              width: 120,
              cellRenderer: percentCellRenderer,
              filter: ProficiencyFilter
            }
          ]
        },
        {
          headerName: "Contact",
          children: [
            {
              headerName: "Mobile",
              field: "mobile",
              width: 150,
              filter: "text"
            },
            {
              headerName: "Land-line",
              field: "landline",
              width: 150,
              filter: "text"
            },
            {
              headerName: "Address",
              field: "address",
              width: 500,
              filter: "text"
            }
          ]
        }
      ];
    },
    pad(num, totalStringSize) {
      let asString = num + "";
      while (asString.length < totalStringSize) asString = "0" + asString;
      return asString;
    },

    calculateRowCount() {
      if (this.api && this.rowData) {
        let model = this.gridOptions.api.getModel();
        let totalRows = this.rowData.length;
        let processedRows = model.getRowCount();
        this.rowCount =
          processedRows.toLocaleString() + " / " + totalRows.toLocaleString();
      }
    },
    onModelUpdated() {
      this.calculateRowCount();
    },
    onGridReady(params) {
      this.api = params.api;
      this.calculateRowCount();
      this.api.sizeColumnsToFit();
    },
  },
  beforeMount() {
    this.gridOptions = {};
    this.gridOptions.frameworkComponents = { agDateInput: DateComponent };
    this.createRowData();
    this.createColumnDefs();
  }
};

function skillsCellRenderer(params) {
  let data = params.data;
  let skills = [];
  RefData.IT_SKILLS.forEach( skill => {
    if (data && data.skills && data.skills[skill]) {
      skills.push(
        '<img src="https://www.ag-grid.com/example-assets/skills/' +
          skill +
          '.png" width="16px" title="' +
          skill +
          '" />'
      );
    }
  });
  return skills.join("");
}

function countryCellRenderer(params) {
  let flag =
    "<img border='0' width='15' height='10' style='margin-bottom: 2px' src='https://www.ag-grid.com/example-assets/flags/" +
    RefData.COUNTRY_CODES[params.value] +
    ".png'>";
  return flag + " " + params.value;
}

function createRandomPhoneNumber() {
  let result = "+";
  for (let i = 0; i < 12; i++) {
    result += Math.round(Math.random() * 10);
    if (i === 2 || i === 5 || i === 8) {
      result += " ";
    }
  }
  return result;
}

function percentCellRenderer(params) {
  let value = params.value;

  let eDivPercentBar = document.createElement("div");
  eDivPercentBar.className = "div-percent-bar";
  eDivPercentBar.style.width = value + "%";
  if (value < 20) {
    eDivPercentBar.style.backgroundColor = "red";
  } else if (value < 60) {
    eDivPercentBar.style.backgroundColor = "#ff9900";
  } else {
    eDivPercentBar.style.backgroundColor = "#00A000";
  }

  let eValue = document.createElement("div");
  eValue.className = "div-percent-value";
  eValue.innerHTML = value + "%";

  let eOuterDiv = document.createElement("div");
  eOuterDiv.className = "div-outer-div";
  eOuterDiv.appendChild(eValue);
  eOuterDiv.appendChild(eDivPercentBar);

  return eOuterDiv;
}
</script>

<style scoped>
.ag-theme-alpine-dark  >>> * {
  font-size: 12px;
}

.ag-theme-alpine-dark >>> .ag-checkbox-input-wrapper {
  width: 12px;
  height: 12px;
  line-height: 12px;
}

.ag-theme-alpine-dark >>> .ag-root-wrapper {
  border-radius: 8px;
  font-size: 12px;
}
.ag-theme-alpine-dark >>> .ag-cell {
  padding-top: 2px !important;
  padding-bottom: 2px !important;
}

.ag-theme-alpine-dark >>> label {
  font-weight: normal !important;
}

.ag-theme-alpine-dark >>> .div-percent-bar {
  display: inline-block;
  height: 100%;
  position: relative;
  z-index: 0;
}

.ag-theme-alpine-dark >>> .div-percent-value {
  position: absolute;
  padding-left: 4px;
  font-weight: bold;
  font-size: 13px;
  z-index: 100;
}

.ag-theme-alpine-dark >>> .div-outer-div {
  display: inline-block;
  height: 100%;
  width: 100%;
}

.ag-theme-alpine-dark >>> .ag-menu {
  z-index: 200;
}

.ag-theme-alpine-dark >>> .toolbar button {
  margin-left: 5px;
  margin-right: 5px;
  padding: 2px;
}
</style>
