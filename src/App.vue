<template>
  <div id="app">
    <b-card title="Import QR Code Text File">
      <b-form-file
        v-model="file1"
        :state="Boolean(file1)"
        placeholder="Choose a file or drop it here"
        drop-placeholder="Drop file here..."
      ></b-form-file>
    </b-card>
    <b-card title="Preview Data">
      <b-form-input v-model="filter" placeholder="Search data"></b-form-input>
      <b-table
        responsive
        type="search"
        :filter="filter"
        :items="formatted"
      ></b-table>
      <b-button variant="success" @click="download">Download</b-button>
    </b-card>
  </div>
</template>

<script>
import XLSX from "xlsx";

export default {
  name: "App",
  data() {
    return {
      file1: null,
      filter: "",
      formatted: [],
    };
  },
  watch: {
    file1(val) {
      const fr = new FileReader();
      const app = this;
      fr.readAsText(val);
      fr.onload = function (e) {
        app.formatted.length = 0
        const qrCodes = e.target.result.split("\r\n");
        const groupLength = qrCodes.length > 0 ? qrCodes[0].length - 1 : 0;
        const groups = qrCodes
          .reduce((acc, curr) => {
            if (!acc.includes(curr.substring(0, groupLength))) {
              acc.push(curr.substring(0, groupLength));
            }
            return acc;
          }, [])
          .filter((g) => g !== "")
          .sort()
          .map((g) => {
            return qrCodes
              .filter((c) => c.substring(0, groupLength) === g)
              .sort();
          });

        const totRows = 16;

        for (let i = 0; i < totRows; i++) {
          const row = [];

          for (let j = 0; j < groups.length; j++) {
            row.push(groups[j][i] || "");
          }

          app.formatted.push(row);
        }
      };
    },
  },
  methods: {
    download() {
      console.log("Download:", this.formatted);
      const worksheet = XLSX.utils.aoa_to_sheet(this.formatted);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Sheet1");
      XLSX.writeFile(workbook, "output.xlsx");
    },
  },
};
</script>

<style>
</style>
