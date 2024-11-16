<template>
  <div id="app">
    <DataTable
        apiBaseUrl="https://jsonplaceholder.typicode.com/users"
        :columns="columns"
        :pageSize="3"
        @row-dblclick="showRowDetails"
    />

    <!-- Modal -->
    <div
        class="modal fade"
        id="detailsModal"
        tabindex="-1"
        aria-labelledby="detailsModalLabel"
        aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="detailsModalLabel">Row Details</h5>
            <button
                type="button"
                class="btn-close"
                data-bs-dismiss="modal"
                aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <ul>
              <li v-for="(value, key) in selectedRow" :key="key">
                <strong>{{ key }}:</strong> {{ value }}
              </li>
            </ul>
          </div>
          <div class="modal-footer">
            <button
                type="button"
                class="btn btn-secondary"
                data-bs-dismiss="modal"
            >
              Close
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import DataTable from "./components/Datatable.vue";

export default {
  name: "App",
  components: {
    DataTable,
  },
  data() {
    return {
      columns: [
        {key: "id", label: "ID"},
        {key: "name", label: "Name"},
        {key: "email", label: "Email"},
      ],
      selectedRow: null,
    };
  },
  methods: {
    showRowDetails(row) {
      this.selectedRow = row;
      const modal = new bootstrap.Modal(document.getElementById("detailsModal"));
      modal.show();
    },
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
}
</style>
