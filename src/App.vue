<script>
const TIME_TYPE = "TIME_TYPE";
const NUMBER_TYPE = "NUMBER_TYPE";

const convertMinsToHrsMins = (minutes) => {
  var h = Math.floor(minutes / 60);
  var m = minutes % 60;
  h = h < 10 ? "" + h : h;
  m = m < 10 ? "0" + m : m;
  return h + ":" + m;
};

const withoutProperty = (obj, property) => {
  const { [property]: unused, ...rest } = obj;

  return rest;
};

const rowTemplate = (id, type, value) => ({
  id,
  type, // 'TIME_TYPE' | 'NUMBER_TYPE'
  value,
});

export default {
  data() {
    return {
      rows: {},
    };
  },

  methods: {
    addTimeRow() {
      const id = Date.now();
      this.rows[id] = rowTemplate(id, TIME_TYPE, { start: null, end: null });
    },

    addNumberRow() {
      const id = Date.now();
      this.rows[id] = rowTemplate(id, NUMBER_TYPE, 0);
    },

    deleteRow(id) {
      this.rows = withoutProperty(this.rows, id);
    },

    saveData() {
      // if (Object.values(this.rows).length === 0) {
      localStorage.setItem("k-app", JSON.stringify(this.rows));
      // }
    },

    loadData() {
      try {
        this.rows = JSON.parse(localStorage.getItem("k-app"));
      } catch (error) {}
    },

    clearAll() {
      if (confirm("Are you sure you want to delete all data?")) {
        localStorage.clear();
        this.rows = {};
      }
    },

    rowSum(row) {
      let d1 = new Date(1600000000000);
      let d2 = new Date(1600000000000);

      const hour1 = row.value.start ? row.value.start.split(":")[0] : 0;
      const minutes1 = row.value.start ? row.value.start.split(":")[1] : 0;

      d1.setHours(hour1);
      d1.setMinutes(minutes1);

      const hour2 = row.value.end ? row.value.end.split(":")[0] : 0;
      const minutes2 = row.value.end ? row.value.end.split(":")[1] : 0;

      d2.setHours(hour2);
      d2.setMinutes(minutes2);

      const finalMins = (d2 - d1) / 1000 / 60;
      return convertMinsToHrsMins(finalMins);
    },
  },

  computed: {
    totalTime() {
      const totalMins = Object.values(this.rows)
        .map((row) => {
          if (row.type == NUMBER_TYPE) {
            const val = parseInt(row.value);
            if (isNaN(val)) {
              return 0;
            }
            return val;
          }

          let d1 = new Date(1600000000000);
          let d2 = new Date(1600000000000);

          const hour1 = row.value.start ? row.value.start.split(":")[0] : 0;
          const minutes1 = row.value.start ? row.value.start.split(":")[1] : 0;

          d1.setHours(hour1);
          d1.setMinutes(minutes1);

          const hour2 = row.value.end ? row.value.end.split(":")[0] : 0;
          const minutes2 = row.value.end ? row.value.end.split(":")[1] : 0;

          d2.setHours(hour2);
          d2.setMinutes(minutes2);

          const finalMins = (d2 - d1) / 1000 / 60;
          return finalMins;
        })
        .reduce((accumulator, currentValue) => accumulator + currentValue, 0);

      this.saveData();
      return convertMinsToHrsMins(totalMins);
    },
  },

  created() {
    if (!localStorage.getItem("k-app")) {
      localStorage.setItem("k-app", {});
    }
    this.loadData();
  },
};
</script>

<template>
  <div class="content">
    <h1>Kilory's App 🧮🎉</h1>

    <h3>
      Total Time:
      <b>{{ totalTime }}</b>
    </h3>
    <button @click="clearAll">Clear All</button>

    <hr />
    <table>
      <tr>
        <td>Start Time</td>
        <td>End Time</td>
        <td>Total</td>
        <td></td>
      </tr>
      <tr v-for="row in rows">
        <template v-if="row.type === 'TIME_TYPE'">
          <td>
            <input
              type="time"
              :value="this.rows[row.id].value.start"
              @change="(e) => (this.rows[row.id].value.start = e.target.value)"
            />
          </td>
          <td>
            <input
              type="time"
              :value="this.rows[row.id].value.end"
              @change="(e) => (this.rows[row.id].value.end = e.target.value)"
            />
          </td>
          <td>{{ rowSum(row) }}</td>
          <td><button @click="deleteRow(row.id)">Delete Row</button></td>
        </template>

        <template v-if="row.type === 'NUMBER_TYPE'">
          <td>
            <input
              type="number"
              style="width: 126px"
              :value="this.rows[row.id].value"
              @input="(e) => (this.rows[row.id].value = e.target.value)"
            />
          </td>
          <td></td>
          <td></td>
          <td><button @click="deleteRow(row.id)">Delete Row</button></td>
        </template>
      </tr>
    </table>

    <hr />
    <div>
      <button @click="addTimeRow">Add Time Row +</button>
      <button @click="addNumberRow">Add Minutes Row +</button>
    </div>
  </div>
</template>

<style>
.content {
  max-width: 700px;
  margin: auto;
}
</style>
