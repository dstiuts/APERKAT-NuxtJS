<template>
  <div class="">
    <!-- <button @click="getrkat">getrkat</button> -->
    <div class="row">
      <!-- Area Chart -->
      <div class="col-xl-12 col-lg-12">
        <div class="card shadow mb-4">
          <!-- Card Header - Dropdown -->
          <div
            class="card-header py-3 d-flex flex-row align-items-center justify-content-between"
          >
            <h6 class="m-0 font-weight-bold text-primary">Form Ubah RKAT</h6>
          </div>
          <!-- Card Body -->
          <form-rkat />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import rkatform from "~/components/form/rkat.vue";
import { mapActions, mapMutations } from "vuex";

export default {
  async asyncData({ store, params }) {
    await Promise.all([
      store.dispatch("rkat/getrkatID", params.id),
      store.dispatch("rkat/getUser")
    ]);
    return;
  },
  components: {
    "form-rkat": rkatform,
  },
  mounted() {
    // this.SET_IS_AUTH(this.$store.state.auth.loggedIn);
    // this.SET_USER_DATA(this.$store.state.auth.user[0]);
  },
  methods: {
    ...mapMutations(["SET_IS_AUTH", "SET_USER_DATA"]),
    ...mapActions("rkat", ["getrkatID"]),
  }
};
</script>

<style>
</style>