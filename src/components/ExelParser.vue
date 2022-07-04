<template>
  <v-container>
    <!-- region nav-->
    <v-row class="text-center">
      <v-col cols="4">
        <v-file-input
          placeholder="Put Exel file here"
          flat
          hide-details
          accept=".xls"
          @change="onChangeFile"
        />
      </v-col>

      <v-col cols="2">
        <v-checkbox v-model="alternativeTimeFormat" label="Use alternative time format" />
      </v-col>

      <v-col v-if="workLogs.length" cols="4" class="d-flex align-center">
        <v-btn @click="isDialogShow = true">Columns settings</v-btn>

        <columns-dialog
          :is-dialog-show.sync="isDialogShow"
          :date-field="dateField"
          :log-fields="logFields"
          @changeRowVisible="changeRowVisible"
          @changeDateField="changeDateField"
        />

        <v-btn class="ml-4" @click="updateColumnSettings">
          update Column Settings
        </v-btn>
      </v-col>
    </v-row>
    <!-- endregion nav-->

    <!-- region alert -->
    <v-row v-if="!dateField && workLogs.length">
      <v-col cols="12">
        <div class="flex justify-center">
          <v-alert
            dense
            border="left"
            type="warning"
          >
            Specify date field in column settings
          </v-alert>
        </div>
      </v-col>
    </v-row>
    <!-- endregion alert -->

    <!-- region log -->
    <v-row class="text-center">
      <v-col cols="3" v-for="(workLog, i) in workLogs" :key="i">
        <v-card>
          <!-- region log-title -->
          <v-card-title>
            {{ workLog.date }}
            <v-btn class="ml-4" color="primary" @click="onCopy(workLog)">Copy day</v-btn>
            <v-btn v-if="Object.keys(workLog.logs).length > 1" class="ml-4" text @click="workLog.show = !workLog.show">
              <v-icon color="primary">
                {{ workLog.show ? 'mdi-eye-off' : 'mdi-eye' }}
              </v-icon>
            </v-btn>
          </v-card-title>
          <!-- endregion log-title -->

          <v-card-text>
            <!-- region table -->
            <v-simple-table v-for="(logObject, j) in workLog.logs" :key="j" class="text-left">
              <template v-if="j === 0 || workLog.show" v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left">
                      Column
                    </th>
                    <th class="text-left">
                      Value
                    </th>
                  </tr>
                </thead>

                <tbody>
                  <template v-for="(log, key) in logObject">
                    <tr v-if="logFields[key] && (j === 0 || workLog.show)" :key="key">
                      <td>{{ key }}</td>
                      <td>{{ log }}</td>
                    </tr>
                  </template>
                  <tr>
                    <td>Action</td>
                    <td><v-btn width="100%" text color="primary" @click="onCopy(workLog, logObject)">Copy log</v-btn></td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
            <!-- endregion log-table -->
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <!-- endregion log -->
  </v-container>
</template>

<script>
import { read, utils } from 'xlsx';
import copy from 'copy-text-to-clipboard';
import ColumnsDialog from '@/components/ColumnsDialog.vue';

export default {
  name: 'ExelParser',

  components: { ColumnsDialog },

  data() {
    return {
      workLogs: [],
      alternativeTimeFormat: true,
      dateField: '',
      logFields: {},
      isDialogShow: false,
      file: null,
    };
  },

  created() {
    this.init();
  },

  methods: {
    async init() {
      this.logFields = await JSON.parse(localStorage.getItem('logFieldsState')) || {};
      this.alternativeTimeFormat = await JSON.parse(localStorage.getItem('alternativeTimeFormat'));
      this.dateField = localStorage.getItem('dateField') || '';
    },

    updateColumnSettings() {
      Object.keys({ ...this.workLogs[0].logs[0] })
        .forEach((key) => {
          console.log(this.logFields[key]);
          this.logFields[key] = this.logFields[key] === undefined ? true : this.logFields[key];
        });
    },

    changeRowVisible({ key, value }) {
      this.logFields[key] = value;

      localStorage.setItem('logFieldsState', JSON.stringify(this.logFields));
    },
    changeDateField(key) {
      if (!this.dateField) {
        this.dateField = key;
        this.onChangeFile(this.file);
      }

      this.dateField = key;

      localStorage.setItem('dateField', key.toString());
    },

    dateConvert(date) {
      let convertedDate = date;
      if (this.alternativeTimeFormat) {
        convertedDate = new Date(Math.round((date - (25568 + 1)) * 86400 * 1000));
      }
      convertedDate = new Date(convertedDate).setHours(0, 0, 0, 0);
      return new Date(convertedDate).toDateString();
    },

    onChangeFile(file) {
      if (!file) {
        this.file = file;
        return;
      }
      this.file = file;

      const reader = new FileReader();

      reader.onload = (e) => {
        const data = e.target.result;
        const workbook = read(data, {
          type: 'binary',
        });

        workbook.SheetNames.forEach((sheetName) => {
          this.workLogs = [];
          const rows = utils.sheet_to_row_object_array(workbook.Sheets[sheetName]);
          rows.splice(0, 1);
          rows.splice(rows.length - 1, 1);

          rows.forEach((row) => {
            const currentWorkLog = this.workLogs
              .find((workLog) => workLog.date === this.dateConvert(row[this.dateField]));

            if (currentWorkLog) {
              currentWorkLog.logs.push(row);
            } else {
              this.workLogs.push({
                date: this.dateConvert(row[this.dateField]),
                logs: [row],
                show: false,
              });
            }
          });
          if (this.workLogs.length) {
            this.workLogs.sort((a, b) => new Date(a.date) - new Date(b.date));
            this.updateColumnSettings();
          }
        });
      };

      reader.onerror = (err) => {
        console.log(err);
      };

      reader.readAsBinaryString(this.file);

      if (this.dateField) this.file = null;
    },

    onCopy(workLog, logObject) {
      let logString = '';
      const formatLog = (log) => Object.entries(log)
        .forEach(([key, value]) => {
          if (this.logFields[key]) {
            logString += `${value} \n`;
          }
        });

      if (logObject) formatLog(logObject);
      else {
        workLog.logs.forEach((log, i) => {
          formatLog(log);
          if (i === workLog.logs.length - 2) logString += '\n';
        });
      }

      copy(logString);
    },
  },

  watch: {
    alternativeTimeFormat(v) {
      localStorage.setItem('alternativeTimeFormat', v.toString());
    },
  },
};
</script>
