<template>
  <div class="row">
    <!-- Area Chart -->
    <div class="col-xl-12 col-lg-12">
      <div class="card shadow mb-4">
        <!-- Card Header - Dropdown -->
        <div
          class="
            card-header
            py-3
            d-flex
            flex-row
            align-items-center
            justify-content-between
          "
        >
          <h6 class="m-0 font-weight-bold text-primary">Pengajuan</h6>
        </div>
        <!-- Card Body -->
        <div class="card-body">
          <custom-table
            :items="items"
            :fields="fields"
            :html="key"
            :actions="actions"
            ref="table"
          >
            <template v-slot:fullname="row">
              <p>
                {{ row.item.fullname | capitalize }}
              </p>
            </template>
            <template v-slot:nama_struktur="row">
              <p
                v-if="
                  row.item.nama_struktur !== '0' &&
                  row.item.nama_struktur_child1 == '0' &&
                  row.item.nama_struktur_child2 == '0'
                "
                class="text-uppercase"
              >
                {{ row.item.nama_struktur }}
              </p>
              <p
                v-if="
                  row.item.nama_struktur !== '0' &&
                  row.item.nama_struktur_child1 !== '0' &&
                  row.item.nama_struktur_child2 == '0'
                "
                class="text-uppercase"
              >
                {{ row.item.nama_struktur }}
              </p>
              <p
                v-if="
                  row.item.nama_struktur !== '0' &&
                  row.item.nama_struktur_child1 !== '0' &&
                  row.item.nama_struktur_child2 !== '0'
                "
                class="text-uppercase"
              >
                {{ row.item.nama_struktur_child1 }}
              </p>
            </template>
            <template v-slot:validasi_status="row">
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
              <p v-if="row.item.validasi_status == 2">
                <b-badge variant="success"
                  >Diterima: {{ row.item.nama_status }}</b-badge
                >
              </p>
              <p v-if="row.item.validasi_status == 3">
                <b-badge variant="success"
                  >Pencairan: {{ row.item.nama_status }}</b-badge
                >
              </p>
            </template>
            <template v-slot:created_at="row">
              <p>{{ row.item.created_at | convertDate }}</p>
            </template>
            <template v-slot:actions="row">
              <NuxtLink
                class="btn btn-sm btn-outline-info mt-1"
                :to="'edit/' + row.item.id_pengajuan"
                :key="'edit' + row.index"
                >Detail</NuxtLink
              >
              <button
                class="btn btn-sm btn-outline-danger mt-1"
                @click="destroypengajuan(row.item.id_pengajuan)"
              >
                Hapus
              </button>
            </template>
          </custom-table>
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
      store.dispatch(
        "subordinate/getpengajuan",
        store.$auth.$state.user[0].id_user
      ),
    ]);
    return;
  },
  data() {
    return {
      key: "id_pengajuan",
      actions: [
        {
          name: "Tambah",
          type: "link",
          link: "/pengajuan/subordinate/add",
          color: "btn btn-sm btn-outline-primary mt-1 ml-2",
        },
        {
          name: "Reset",
          type: "func",
          func: "reset",
          color: "btn btn-sm btn-outline-primary mt-1 ml-2",
        },
        {
          name: "Select All",
          type: "func",
          func: "selectAll",
          color: "btn btn-sm btn-outline-primary mt-1 ml-2",
        },
        {
          name: "Clear Selected",
          type: "func",
          func: "clearSelected",
          color: "btn btn-sm btn-outline-primary mt-1 ml-2",
        },
        {
          name: "Print",
          type: "func",
          func: "print",
          color: "btn btn-sm btn-outline-primary mt-1 ml-2",
        },
        {
          name: "Print Selected",
          type: "func",
          func: "printSelected",
          color: "btn btn-sm btn-outline-primary mt-1 ml-2",
        },
        {
          name: "Delete Selected",
          type: "func",
          func: "deleteSelected",
          color: "btn btn-sm btn-outline-primary mt-1 ml-2",
        },
      ],
      fields: [
        { key: "fullname", label: "User" },
        { key: "kode_rkat", label: "Kode RKAT " },
        { key: "nama_struktur", label: "Fakultas/Unit Pelaksana" },
        { key: "validasi_status", label: "Status Pengajuan" },
        { key: "created_at", label: "Waktu Pengajuan" },
        "actions",
      ],
      items: [],
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
  mounted() {
    this.items = this.pengajuan;
  },
  methods: {
    ...mapActions("subordinate", ["getpengajuan", "deletepengajuan"]),
    rowClass(item, type) {
      if (!item || type !== "row") return;
      if (item.status_message === 0) return "table-success";
    },
    destroypengajuan(row) {
      this.$swal({
        title: "Warning!",
        text: "Yakin menghapus pengajuan?",
        icon: "warning",
        width: 300,
        showCancelButton: true,
        confirmButtonColor: "#d33",
        cancelButtonColor: "#3085d6",
        confirmButtonText: "OK, Hapus!",
      }).then((result) => {
        if (result.isConfirmed) {
          this.deletepengajuan(row.item.id_pengajuan)
            .then(() => {
              this.$swal({
                width: 300,
                icon: "success",
                title: "Congrats!",
                text: "Data telah dihapus",
              });
              this.reload();
            })
            .catch(() => {
              this.$swal({
                width: 300,
                icon: "error",
                title: "Oops...",
                text: "Server error atau cek koneksi anda",
              });
            });
        }
      });
    },
    deleteAll() {
      this.$swal({
        title: "Warning!",
        text: "Hapus semua pengajuan?",
        icon: "warning",
        width: 300,
        showCancelButton: true,
        confirmButtonColor: "#d33",
        cancelButtonColor: "#3085d6",
        confirmButtonText: "OK!",
      }).then((result) => {
        if (result.isConfirmed) {
          this.$axios
            .get(`/pengajuan/destroy/${this.$store.state.auth.user[0].id_user}`)
            .then(async () => {
              await this.$swal({
                width: 300,
                icon: "success",
                title: "Congrats!",
                text: "Semua pengajuan telah dihapus",
              });
              await this.reload();
            })
            .catch(() => {
              this.$swal({
                width: 300,
                icon: "error",
                title: "Oops...",
                text: "Cek server atau koneksi anda",
              });
            });
        }
      });
    },
    print(params) {
      this.$axios
        .post("/pengajuan/pdfByUSer", params, {
          responseType: "blob",
        })
        .then((res) => {
          const url = window.URL.createObjectURL(new Blob([res.data]));
          const link = document.createElement("a");
          link.href = url;
          link.setAttribute("download", "RKAT.pdf");
          document.body.appendChild(link);
          link.click();
          this.$swal({
            width: 300,
            icon: "success",
            title: "Congrats!",
            text: "RKAT telah dihapus",
          });
        });
    },
    deleteRows(params) {
      this.$swal({
        title: "Warning!",
        text: "Yakin menghapus pengajuan terpilih?",
        icon: "warning",
        width: 300,
        showCancelButton: true,
        confirmButtonColor: "#d33",
        cancelButtonColor: "#3085d6",
        confirmButtonText: "OK",
      }).then((result) => {
        if (result.isConfirmed) {
          this.$axios
            .post("/pengajuan/deleteRows", params)
            .then(async () => {
              await this.$swal({
                width: 300,
                icon: "success",
                title: "Congrats!",
                text: "Pengajuan telah dihapus",
              });
              await this.reload();
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
    async reload() {
      await this.$nuxt.refresh();
      await this.$refs.table.reload(this.pengajuan);
    },
  },
};
</script>

<style>
</style>