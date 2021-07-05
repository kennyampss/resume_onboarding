<template>
  <div>
    <div class="text-center">
      <h1>
        {{ title }}
      </h1>
    </div>
    <div class="d-flex ml-4" style="padding: 40px 0px 30px;">
      <v-btn color="primary" @click="add">Add</v-btn>
    </div>
    <div style="margin: 0 auto; width: 95%; padding-bottom: 40px;">
      <div
        class="d-flex align-center"
        v-for="(comment, i) in form.comments"
        :key="`comment-${i}`"
      >
        <v-card class="my-2 pa-2" style="flex: 1">
          <div class="d-flex align-center">
            <div style="text-align: left; flex: 1">{{ comment.company }}</div>
            <div style="text-align: left; flex: 1">{{ comment.position }}</div>
            <div style="text-align: left; flex: 1">{{ comment.date_range }}</div>
            <div style="text-align: left; flex: 3">
              {{ comment.responsibility }}
            </div>
          </div>
        </v-card>
        <v-btn @click="editComment(comment)" class="ma-2" text icon color="red">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn @click="removeComment(i)" class="ma-2" text icon color="red">
          <v-icon>mdi-delete</v-icon>
        </v-btn>
      </div>
    </div>
    <v-dialog v-model="showEditor" width="500">
      <v-card v-if="showEditor">
        <v-card-title class="text-h5 grey lighten-2"> Experience </v-card-title>

        <v-card-text>
          <v-form ref="form-comment" v-model="validComment">
            <v-text-field
              v-model="formComment.company"
              :rules="[rules.required]"
              label="Company"
            ></v-text-field>
            <v-text-field
              v-model="formComment.position"
              :rules="[rules.required]"
              label="Position"
            ></v-text-field>
            <v-text-field
              v-model="formComment.date_range"
              :rules="[rules.required]"
              label="Date Range"
            ></v-text-field>
            <v-textarea
              v-model="formComment.responsibility"
              :rules="[rules.required]"
              label="Responsibilities"
            ></v-textarea>
          </v-form>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="validateComment()">
            {{ formComment.id ? "Update" : "Create" }}
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
      validComment: true,
      showEditor: false,
      rules: {
        required: (value) => !!value || "Required.",
      },
      defaultFormValue: {
        company: "",
        position: "",
        date_range: "",
        responsibility: "",
      },
      formComment: {
        company: "",
        position: "",
        date_range: "",
        responsibility: "",
      },
    };
  },

  computed: {
    lastIdComment() {
      if(!last(this.form.comments)) return 1;

      return last(this.form.comments).id;
    },
  },

  methods: {
    add() {
      this.showEditor = true;
    },
    editComment(val) {
      this.formComment = { ...val };
      this.showEditor = true;
    },
    removeComment(i) {
      this.form.comments.splice(i, 1);
    },
    validateComment() {
      if (this.$refs["form-comment"].validate()) {
        if (!this.formComment.id) {
          this.formComment.id = this.lastIdComment + 1;
          this.form.comments.push(this.formComment);
        } else {
          let _findIndex = findIndex(
            this.form.comments,
            (e) => e.id == this.formComment.id
          );
          if (_findIndex > -1) {
            this.form.comments.splice(_findIndex, 1, this.formComment);
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
    resetComment() {
      this.formComment = { ...this.defaultFormValue };
    },
  },

  watch: {
    showEditor(v) {
      if (!v) {
        this.resetComment();
      }
    },
  },
};
</script>

<style>
</style>