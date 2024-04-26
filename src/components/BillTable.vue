<template>
  <div class="q-pa-md">
    <q-table
      flat
      bordered
      :title="eventName != '' ? eventName : 'Untitled Event'"
      ref="billTable"
      table-class="text-right"
      rows-per-page-options="0"
      :rows="rows"
      :columns="columns"
      row-key="id"
      selection="multiple"
      v-model:selected="selected"
    >
      <!-- Name column -->
      <template v-slot:body-cell-name="props">
        <q-td :props="props">
          <q-input
            filled
            v-model.number="props.row[props.col.name]"
            input-class="text-right"
            dense
            borderless
            @update:model-value="(value) => updateName(value, props.row.id)"
        /></q-td>
      </template>
      <!-- Individual items column-->
      <template v-slot:body-cell="props">
        <q-td :props="props" v-if="props.col.editable">
          <q-input
            filled
            v-model.number="props.row[props.col.name]"
            input-class="text-right"
            dense
            borderless
            @update:model-value="
              (value) => updateTotalCost(props.row.id)
            "
        /></q-td>
      </template>

      <!-- Cost column-->
      <template v-slot:body-cell-cost="props">
        <q-td :props="props" v-if="this.columns.length < 4">
          <q-input
            filled
            v-model.number="props.row[props.col.name]"
            input-class="text-right"
            dense
            borderless
            @update:model-value="(value) => updateRow(value, props.row.id)"
            lazy-rules
            :rules="[(val) => !isNaN(val) || 'Please type a number']"
        /></q-td>
        <q-td v-else>{{ props.row.cost }}</q-td>
      </template>
      <!-- How much to pay column -->
      <template v-slot:body-cell-pay="props">
        <q-td :props="props">{{ props.row.pay }}</q-td>
      </template>
      <!-- Final row which shows totals -->
      <template v-slot:bottom-row>
        <q-tr>
          <q-td v-for="index in columns.length - 1" :key="index"></q-td>
          <q-td v-bind:class="calculatedSubtotal != subtotal ? 'text-red' : 'text-green'"
            >Calculated subtotal: ${{ calculatedSubtotal }}</q-td
          >
          <q-td v-bind:class="calculatedTotal != total ? 'text-red' : 'text-green'"
            >Calculated total: ${{ calculatedTotal }}</q-td
          >
        </q-tr>
        <q-tr>
          <q-td v-for="index in columns.length - 1" :key="index"></q-td>
          <q-td>Subtotal: ${{ subtotal }}</q-td>
          <q-td>Total: ${{ total }}</q-td>
        </q-tr>
      </template>

      <!-- Bottom of table -->
      <template v-slot:bottom>
        <q-btn
          flat
          dense
          color="primary"
          :disable="loading"
          label="Add row"
          @click="addRow"
        ></q-btn>
        <q-btn
          class="on-right"
          flat
          dense
          color="primary"
          :disable="loading"
          label="Remove row"
          @click="removeRow"
        ></q-btn>
        <q-btn
          class="on-right"
          flat
          dense
          color="primary"
          :disable="loading"
          label="Add column"
          @click="newColumnDialog = true"
        ></q-btn>

      </template>
    </q-table>

    <q-dialog v-model="newColumnDialog" persistent>
      <q-card style="min-width: 350px">
        <q-card-section>
          <div class="text-h6">New Column</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input
            dense
            v-model="columnName"
            autofocus
            @keyup.enter="addColumn"
            placeholder="Name of column"
          />
          <q-input
            dense
            v-model="columnCost"
            autofocus
            @keyup.enter="addColumn"
            placeholder="Total cost of item"
          />
          Who to split this among
          <q-checkbox
            v-for="index in rows.length"
            :key="index"
            v-model="rows[Number(index - 1)].checked"
            >{{ rows[Number(index - 1)].name }}</q-checkbox
          >
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancel" v-close-popup />
          <q-btn flat label="Confirm" v-close-popup @click="addColumn" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { defineComponent } from 'vue';

export default defineComponent({
  props: {
    eventName: {
      default: '',
      type: String,
    },
    numPeople: {
      type: String,
    },
    subtotal: {
      default: '0',
      type: String,
    },
    total: {
      default: '0',
      type: String,
    },
  },
  data() {
    return {
      rows: [],
      columns: [
        { name: 'name', field: 'name', label: 'Name' },
        { name: 'cost', field: 'cost', label: 'Cost' },
        { name: 'pay', field: 'pay', label: 'How much to pay' },
      ],
      selected: [],
      newColumnDialog: false,
      columnName: '',
    };
  },
  computed: {
    calculatedSubtotal() {
      var subtotal = this.rows.reduce((n, { cost }) => n + Number(cost), 0);
      return this.round(subtotal);
    },
    calculatedTotal() {
      var total = this.rows.reduce((n, { pay }) => n + Number(pay), 0);
      return this.round(total);
    },
  },

  methods: {
    updateRow(newValue, id) {
      this.rows[id].pay =
        this.round((newValue / this.subtotal) * this.total);
    },
    updateName(newValue, id) {
      this.rows[id].name = newValue;
    },
    updateTotalCost(id) {
      var sum = 0;
      for (const property in this.rows[id]) {
        if (property.startsWith('item')) {
          sum += Number(this.rows[id][property]);
        }
      }
      this.rows[id].cost = this.round(sum);
      this.rows[id].pay =
        this.round((sum / this.subtotal) * this.total);

    },
    addRow() {
      this.rows.push({
        id: this.rows.length,
        name: '',
        cost: 0,
        pay: 0,
        checked: true,
      });
    },
    removeRow() {
      console.log(this.selected);
    },
    addColumn() {
      var id = this.columns.length - 2;
      this.columns.splice(id, 0, {
        name: 'item' + id,
        label: this.columnName,
        editable: true,
      });
      var cost = 0;
      var numPeople = this.rows.reduce(
        (n, { checked }) => (checked ? 1 : 0) + n,
        0
      );
      if (this.columnCost != '') {
        cost = this.round(Number(this.columnCost) / numPeople)
      }

      for (var i = 0; i < this.rows.length; i++) {
        this.rows[i]['item' + id] = this.rows[i].checked ? cost : 0;
        this.updateTotalCost(i);
      }
      this.columnName = '';
      this.columnCost = '';
      this.newColumnDialog = false;
    },

  round(value) {
    return Math.round(Number(value) * 100) / 100
  },
  },
  created() {
    this.rows = [...Array(Number(this.numPeople)).keys()].map(
      (elem, index) => ({ id: index, name: '', cost: 0, pay: 0, checked: true })
    );
  },
});

//TODO: when you create new column, optional cost of item + who's dividing it
</script>
