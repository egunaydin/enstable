<template>
  <div class="datatable-container container position-relative">
    <!-- Loading Spinner -->
    <div v-if="isLoading" class="loading-overlay">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>

    <!-- Search Bar -->
    <div class="row mb-3">
      <div class="col-md-6">
        <input
            v-if="enableSearch"
            v-model="localSearchQuery"
            type="text"
            placeholder="Search..."
            class="form-control"
            @input="fetchData"
        />
      </div>
    </div>

    <!-- Table -->
    <div class="table-responsive">
      <table class="table table-bordered table-hover">
        <thead class="table-light">
        <tr>
          <th
              v-for="column in columns"
              :key="column.key"
              @click="sortBy(column.key)"
              class="sortable"
          >
            {{ column.label }}
            <span v-if="sortKey === column.key">
                {{ sortOrder === 'asc' ? '↑' : '↓' }}
              </span>
          </th>
        </tr>
        </thead>
        <tbody>
        <tr
            v-for="(item, index) in data"
            :key="item.id"
            @dblclick="$emit('row-dblclick', item)"
        >
          <td v-for="column in columns" :key="column.key">
            {{ item[column.key] }}
          </td>
        </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <nav class="d-flex justify-content-center">
      <ul class="pagination">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <button class="page-link" @click="goToPage(currentPage - 1)">
            &laquo;
          </button>
        </li>
        <li
            v-for="page in totalPages"
            :key="page"
            class="page-item"
            :class="{ active: page === currentPage }"
        >
          <button class="page-link" @click="goToPage(page)">
            {{ page }}
          </button>
        </li>
        <li
            class="page-item"
            :class="{ disabled: currentPage === totalPages }"
        >
          <button class="page-link" @click="goToPage(currentPage + 1)">
            &raquo;
          </button>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
export default {
  name: "DataTable",
  props: {
    apiBaseUrl: {
      type: String,
      required: true,
    },
    columns: {
      type: Array,
      required: true,
    },
    pageSize: {
      type: Number,
      default: 5,
    },
    enableSearch: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      data: [],
      totalRows: 0,
      currentPage: 1,
      localSearchQuery: "",
      isLoading: false,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.totalRows / this.pageSize);
    },
  },
  methods: {
    buildUrl() {
      const url = new URL(this.apiBaseUrl);
      url.searchParams.set("_page", this.currentPage);
      url.searchParams.set("_limit", this.pageSize);
      if (this.localSearchQuery) {
        url.searchParams.set("q", this.localSearchQuery);
      }
      return url.toString();
    },
    async fetchData() {
      try {
        this.isLoading = true;
        const url = this.buildUrl();
        const response = await fetch(url);
        const total = response.headers.get("x-total-count");
        if (total) this.totalRows = parseInt(total, 10);
        this.data = await response.json();
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false;
      }
    },
    goToPage(page) {
      if (page < 1 || page > this.totalPages) return;
      this.currentPage = page;
      this.fetchData();
    },
    sortBy(key) {
      if (this.sortKey === key) {
        this.sortOrder = this.sortOrder === "asc" ? "desc" : "asc";
      } else {
        this.sortKey = key;
        this.sortOrder = "asc";
      }
    },
  },
  mounted() {
    this.fetchData();
  },
};
</script>

<style scoped>
.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
}

.sortable {
  cursor: pointer;
}

.table-hover tbody tr:hover {
  background-color: #f8f9fa;
}

.pagination .page-item.active .page-link {
  background-color: #007bff;
  border-color: #007bff;
  color: white;
}

.pagination .page-item .page-link {
  color: #007bff;
}
</style>
