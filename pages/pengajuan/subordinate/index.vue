<template>
  <div class="row">
    <!-- Area Chart -->
    <div class="col-xl-12 col-lg-12">
      <div class="card shadow mb-4">
        <!-- Card Header - Dropdown -->
        <div
          class="card-header py-3 d-flex flex-row align-items-center justify-content-between"
        >
          <h6 class="m-0 font-weight-bold text-primary">Pengajuan</h6>
          <div class="dropdown no-arrow">
            <a
              class="dropdown-toggle"
              href="#"
              role="button"
              id="dropdownMenuLink"
              data-toggle="dropdown"
              aria-haspopup="true"
              aria-expanded="false"
            >
              <i class="fas fa-ellipsis-v fa-sm fa-fw text-gray-400"></i>
            </a>
            <div
              class="dropdown-menu dropdown-menu-right shadow animated--fade-in"
              aria-labelledby="dropdownMenuLink"
              style=""
            >
              <div class="dropdown-header">Opsi:</div>
              <NuxtLink class="dropdown-item" to="add"
                >Tambah Pengajuan</NuxtLink
              >
              <NuxtLink class="dropdown-item" to="/rkat/reset"
                >Reset Pengajuan</NuxtLink
              >
            </div>
          </div>
        </div>
        <!-- Card Body -->
        <div class="card-body">
          
          <b-row>
            <b-col sm="5" md="6" class="my-1">
              <b-form-group
                label="Per page"
                label-for="per-page-select"
                label-cols-sm="6"
                label-cols-md="4"
                label-cols-lg="3"
                label-align-sm="right"
                label-size="sm"
                class="mb-0"
              >
                <b-form-select
                  id="per-page-select"
                  v-model="perPage"
                  :options="pageOptions"
                  size="sm"
                ></b-form-select>
              </b-form-group>
            </b-col>
            <b-col lg="6" class="my-1 float-right">
              <b-form-group
                label="Filter"
                label-for="filter-input"
                label-cols-sm="3"
                label-align-sm="right"
                label-size="sm"
                class="mb-0"
              >
                <b-input-group size="sm">
                  <b-form-input
                    id="filter-input"
                    v-model="filter"
                    type="search"
                    placeholder="Type to Search"
                  ></b-form-input>
                </b-input-group>
              </b-form-group>
            </b-col>
          </b-row>
            <!-- sticky-header -->
          <b-table
            responsive
            head-variant="light"
            hover
            id="my-table"
            :items="pengajuan"
            :fields="fields"
            :per-page="perPage"
            :current-page="currentPage"
            :filter="filter"
            show-empty
          >
            <template v-slot:cell(nama_struktur_child1)="row">
              <p
                v-if="row.item.nama_struktur_child1 == 0"
                class="text-uppercase"
              >
                {{ row.item.fullname }}
              </p>
              <p v-else class="text-uppercase">
                {{ row.item.nama_struktur_child1 }}
              </p>
            </template>
            <template v-slot:cell(validasi_status)="row">
              <p v-if="row.item.validasi_status == 0">
                <b-badge variant="danger"
                  >Ditolak: {{ row.item.nama_status }}</b-badge
                >
              </p>
              <p v-if="row.item.validasi_status == 1">
                <b-badge variant="warning"
                  >Input/Revisi: {{ row.item.nama_status }}</b-badge
                >
              </p>
              <p v-if="row.item.validasi_status == 2 && row.item.nama_status !== 'Sekniv'">
                <b-badge variant="success"
                  >Diterima: {{ row.item.nama_status }}</b-badge
                >
              </p>
              <p v-if="row.item.validasi_status == 3">
                <b-badge variant="success"
                  >Pencairan: {{ row.item.nama_status }}</b-badge
                >
              </p>
              <p v-if="row.item.validasi_status == 2 && row.item.nama_status == 'Sekniv'">
                <b-badge variant="success"
                  >Selesai: {{ row.item.nama_status }}</b-badge
                >
              </p>
            </template>
            <template v-slot:cell(actions)="row">
              <NuxtLink
                class="btn-sm btn-warning mb-2"
                :to="'edit/' + row.item.id_pengajuan"
                :key="'edit' + row.index"
                >Detail</NuxtLink
              >
              <button
                class="btn-sm btn-danger mt-2"
                @click="destroypengajuan(row)"
              >
                Hapus
              </button>
            </template>
          </b-table>

          <div class="overflow-auto">
            <b-pagination
              v-model="currentPage"
              :total-rows="rows"
              :per-page="perPage"
              aria-controls="my-table"
            ></b-pagination>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapActions, mapState } from "vuex";

export default {
  async asyncData({ store }) {
    await Promise.all([
      store.dispatch("subordinate/getpengajuan", store.$auth.$state.user[0].id_user),
    ]);
    return;
  },
  data() {
    return {
      fields: [
        { key: "fullname", label: "User" },
        { key: "nama_struktur_child1", label: "Fakultas/Unit Pelaksana" },
        { key: "validasi_status", label: "Status Pengajuan" },
        { key: "created_at", label: "Waktu Pengajuan" },
        "actions",
      ],

      perPage: 5,
      pageOptions: [5, 10, 15, { value: 100, text: "Show a lot" }],
      filter: null,
      currentPage: 1,
      items: this.pengajuan
    };
  },
  computed: {
    ...mapState("subordinate", {
      pengajuan: (state) => state.pengajuan,
    }),
    rows() {
      return this.pengajuan.length;
    },
  },
  mounted() {},
  methods: {
    ...mapActions("subordinate", ["getpengajuan", "deletepengajuan"]),

    destroypengajuan(row) {
      this.$swal({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        width: 300,
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!",
      }).then((result) => {
        if (result.isConfirmed) {
          this.deletepengajuan(row.item.id_pengajuan)
          .then(() => {
            this.$swal({
              width: 300,
              icon: "success",
              title: "Congrats!",
              text: "RKAT data was deleted successfully",
            });
          })
          .catch(() => {
            this.$swal({
              width: 300,
              icon: "error",
              title: "Oops...",
              text: "Please check your server or internet connection",
            });
          });
        }
      });
    },
  },
};
</script>

<style>
</style>