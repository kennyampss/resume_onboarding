<template>
  <div>
    <div class="text-center">
      <h1>
        {{ title }}
      </h1>
    </div>
    <div class="d-flex ml-4" style="padding: 40px 0 30px">
      <v-btn color="primary" @click="add">Add</v-btn>
    </div>
    <div style="margin: 0 auto; width: 95%; padding-bottom:40px;">
      <div
        class="d-flex align-center"
        v-for="(rate, i) in form.rates"
        :key="`rate-${i}`"
        
      >
        <v-card class="my-2 pa-2" style="flex: 1">
          <div class="d-flex align-center">
            <div style="text-align: left; flex: 1">{{ rate.title }}</div>
            <div>
              <v-rating readonly v-model="rate.rate"></v-rating>
            </div>
          </div>
        </v-card>
        <v-btn @click="editRate(rate)" class="ma-2" text icon color="red">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn @click="removeRate(i)" class="ma-2" text icon color="red">
          <v-icon>mdi-delete</v-icon>
        </v-btn>
      </div>
    </div>
    <v-dialog v-model="showEditor" width="500">
      <v-card v-if="showEditor">
        <v-card-title class="text-h5 grey lighten-2"> Skills Rate </v-card-title>

        <v-card-text>
          <v-form ref="form-rate" v-model="validRate">
            <v-text-field
              v-model="formRate.title"
              :rules="[rules.required]"
              label="Technology"
            ></v-text-field>
          </v-form>
          <v-rating v-model="formRate.rate" large hover></v-rating>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="validateRate()">
            {{ formRate.id ? "Update" : "Create" }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { last, findIndex } from "lodash";
export default {
  props: {
    form: Object,
    title: String,
  },

  data() {
    return {
      validRate: true,
      showEditor: false,
      rules: {
        required: (value) => !!value || "Required.",
      },
      defaultFormValue: {
        rate: 0,
        title: "",
      },
      formRate: {
        rate: 0,
        title: "",
      },
    };
  },

  computed: {
    lastIdRate() {
      if(!last(this.form.rates)) return 1;
      
      return last(this.form.rates).id;
    },
  },

  methods: {
    add() {
      this.showEditor = true;
    },
    editRate(val) {
      this.formRate = { ...val };
      this.showEditor = true;
    },
    removeRate(i) {
      this.form.rates.splice(i, 1);
    },
    validateRate() {
      if (this.$refs["form-rate"].validate()) {
        if (!this.formRate.id) {
          this.formRate.id = this.lastIdRate + 1;
          this.form.rates.push(this.formRate);
        } else {
          let _findIndex = findIndex(
            this.form.rates,
            (e) => e.id == this.formRate.id
          );
          if (_findIndex > -1) {
            this.form.rates.splice(_findIndex, 1, this.formRate);
          }
        }
        console.log("create");
        this.showEditor = false;
      } else {
        console.log("error create");
      }
    },
    validate() {
      console.log("proceed");
      this.$emit("next");
    },
    resetRate() {
      this.formRate = { ...this.defaultFormValue };
    },
  },

  watch: {
    showEditor(v) {
      if (!v) {
        this.resetRate();
      }
    },
  },
};
</script>

<style>
</style>