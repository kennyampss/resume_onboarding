<template>
  <div style="height: 100vh; overflow-y: auto" class="d-flex justify-center">
    <!-- <pre>{{ form }}</pre> -->
    <v-card style="width: 35%" class="pa-4">
      <Step1
        ref="step-1"
        :title="'Basic Info'"
        :form="form"
        v-if="step == 1"
        @next="next()"
      ></Step1>
      <Step2
        ref="step-2"
        :title="'Technical Skills'"
        :form="form"
        v-if="step == 2"
        @next="next()"
      ></Step2>
      <Step3
        ref="step-3"
        :title="'Work Experience'"
        :form="form"
        v-if="step == 3"
        @next="next()"
      ></Step3>
      <div class="d-flex justify-end">
        <v-btn v-if="step > 1" @click="back()" dense>Back</v-btn>
        <v-btn
          v-if="step < 3"
          @click="validate()"
          color="primary"
          dense
          class="ml-2"
          >Next</v-btn
        >
        <v-btn
          v-if="step == 3"
          @click="submit()"
          color="primary"
          dense
          class="ml-2"
          >Submit</v-btn
        >
      </div>
    </v-card>
  </div>
</template>

<script>
import Step1 from "./steps/Step1";
import Step2 from "./steps/Step2";
import Step3 from "./steps/Step3";
import axios from "axios";
var pdfMake = require("pdfmake/build/pdfmake.js");
var pdfFonts = require("pdfmake/build/vfs_fonts.js");
pdfMake.vfs = pdfFonts.pdfMake.vfs;

export default {
  components: {
    Step1,
    Step2,
    Step3,
  },

  data() {
    return {
      step: 1,
      myProfile: null,
      form: {
        avatar: null,
        first_name: "",
        last_name: "",
        middle_name: "",
        position: "",
        intro_experienced: "",
        mobile: "",
        email: "",
        address: "",
        birthdate: null,
        rates: [],
        comments: [],
      },
    };
  },

  mounted() {
    this.init();
  },

  methods: {
    init() {
      axios.get("http://localhost:5050/profile").then((res) => {
        this.myProfile = res.data[0];
        this.form = { ...this.form, ...this.myProfile };
      });
    },
    validate() {
      if (!this.$refs[`step-${this.step}`]) return;

      this.$refs[`step-${this.step}`].validate();
    },
    submit() {
      if (this.myProfile) {
        axios.put(`http://localhost:5050/profile/${this.myProfile.id}`, this.form).then(
          (res) => {
            this.createPdf();
            this.step = 1;
          },
          (err) => {
            console.log("error");
          }
        );
      } else {
        axios.post("http://localhost:5050/profile", this.form).then(
          (res) => {
            this.myProfile = res.data
            this.createPdf();
            this.step = 1;
          },
          (err) => {
            console.log("error");
          }
        );
      }
    },
    createPdf() {
      var dd = {
        content: [
          {
            text: "Basic Info:",
            style: "header",
          },
          "\n",
          `Name: ${this.form.first_name} ${this.form.middle_name} ${this.form.last_name}`,
          `Position: ${this.form.position}`,
          `Intro Experience: ${this.form.intro_experienced}`,
          `Contact Number: ${this.form.mobile}`,
          `Email: ${this.form.email}`,
          `Address: ${this.form.address}`,
          `Birthdate: ${this.form.birthdate}`,
          "\n",
        ],
        styles: {
          header: {
            fontSize: 18,
            bold: true,
          },
        },
      };
      this.checkAvatar(dd);
      this.checkRates(dd);
      this.checkComments(dd);
      pdfMake.createPdf(dd).open();
    },
    checkAvatar(dd) {
      if (this.form.avatar) {
        dd.content.unshift({
          image: this.form.avatar,
          fit: [100, 100],
        });
      }
    },
    checkRates(dd) {
      if (this.form.rates && this.form.rates.length > 0) {
        dd.content.push({
          text: "Skills Rate:",
          style: "header",
        });
        dd.content.push({
          text: "\n",
        });
        this.form.rates.forEach((rate) => {
          let _columnStar = {
            columns: [
              {
                width: "auto",
                text: `${rate.title}:  `,
              },
            ],
          };
          for (let i = 0; i < rate.rate; i++) {
            _columnStar.columns.push({
              svg: this.starSvg,
              width: 12,
              height: 12,
            });
          }
          dd.content.push(_columnStar);
        });
      }
    },
    checkComments(dd) {
      if (this.form.comments && this.form.comments.length > 0) {
        dd.content.push({
          text: "\n",
        });
        dd.content.push({
          text: "Work Experience:",
          style: "header",
        });
        dd.content.push({
          text: "\n",
        });
        this.form.comments.forEach((comment) => {
          dd.content.push(`Company: ${comment.company}`);
          dd.content.push(`Position: ${comment.position}`);
          dd.content.push(`Date Range: ${comment.date_range}`);
          dd.content.push(`Responsibility: ${comment.responsibility}`);
          dd.content.push({text: "\n",});
        });
      }
    },
    next() {
      if (this.step > 2) return;

      this.step++;
    },
    back() {
      if (this.step < 1) return;

      this.step--;
    },
  },

  computed: {
    starSvg() {
      return '<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 51 48"><path fill="red" d="m25,1 6,17h18l-14,11 5,17-15-10-15,10 5-17-14-11h18z"/></svg>';
    },
  },
};
</script>

<style>
</style>