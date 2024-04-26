<template>
  <BillTable
    :eventName="eventName"
    :numPeople="numPeople"
    :subtotal="subtotal"
    :total="total"
    class="fixed-center full-width"
    v-if="!showForm"
  ></BillTable>
  <q-card class="q-pa-md fixed-center" v-if="showForm">
    <q-form @submit="onSubmit" @reset="onReset" class="q-gutter-md">
      <q-card-section class="bg-primary">
        <q-input filled v-model="eventName" label="Event Name" />
      </q-card-section>

      <q-input
        filled
        type="number"
        v-model="numPeople"
        label="# of people"
        lazy-rules
        :rules="[
          (val: String) =>
            (val !== null) || 'Please type the number of people',
          (val: String) => Number(val) > 0 || 'Please type a positive integer',
        ]"
      />
      <q-input
        filled
        v-model="subtotal"
        label="Subtotal"
        lazy-rules
        :rules="[
          (val: String) =>
            (val !== null) || 'Please enter the subtotal',
          (val: String) => Number(val) > 0 || 'Please type a positive number',
        ]"
      />
      <q-input
        filled
        v-model="total"
        label="Total"
        lazy-rules
        :rules="[
          (val: String) =>
            (val !== null) || 'Please enter the total',
          (val: String) => Number(val) > 0 || 'Please type a positive number',
        ]"
      />
      <div>
        <q-btn label="Submit" type="submit" color="primary" />
        <q-btn
          label="Reset"
          type="reset"
          color="primary"
          flat
          class="q-ml-sm"
        />
      </div>
    </q-form>
  </q-card>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import BillTable from './components/BillTable.vue';

export default defineComponent({
  components: {
    BillTable,
  },
  data() {
    return {
      eventName: '',
      numPeople: '',
      subtotal: '',
      total: '',
      showForm: true,
    };
  },

  methods: {
    onSubmit(): void {
      this.showForm = false;
    },
    onReset(): void {
      this.eventName = '';
      this.numPeople = '';
      this.subtotal = '';
      this.total = '';
    },
  },
});
</script>
