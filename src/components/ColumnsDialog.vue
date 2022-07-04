<template>
  <v-dialog v-model="computedDialogShow" width="500">
    <v-card>
      <v-card-title>
        Columns settings
      </v-card-title>
      <v-card-text>
        <v-simple-table>
          <thead>
          <tr>
            <th class="text-left">
              Visible
            </th>
            <th class="text-left">
              It is a time
            </th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(value, key) in logFields" :key="key">
            <td>
              <v-checkbox :input-value="value" :label="key" @click="changeRowVisible(key, !value)" />
            </td>
            <td>
              <v-checkbox :input-value="dateField === key" @click="changeDateField(key)" />
            </td>
          </tr>
          </tbody>
        </v-simple-table>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: 'ColumnsDialog',

  props: {
    isDialogShow: {
      type: Boolean,
      required: true,
    },
    logFields: {
      type: Object,
      required: true,
    },
    dateField: {
      type: String,
      required: true,
    },
  },

  computed: {
    computedDialogShow: {
      get() {
        return this.isDialogShow;
      },
      set(v) {
        this.$emit('update:isDialogShow', v);
      },
    },
  },

  methods: {
    changeRowVisible(key, value) {
      this.$emit('changeRowVisible', { key, value });
    },
    changeDateField(key) {
      this.$emit('changeDateField', key);
    },
  },
};
</script>

<style scoped>

</style>
