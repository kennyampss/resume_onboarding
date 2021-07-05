<template>
  <div>
    <input ref="upload-image" type="file" @change="uploadImage" style="display:none" accept="image/*"/>
    <div class="text-center">
      <h1>
        {{ title }}
      </h1>
    </div>
    <v-form ref="form-1" v-model="valid">
      <div class="d-flex justify-center" style="padding: 40px">
        <div style="position: relative; padding: 0 14px">
          <v-avatar size="100">
            <img
              style="background-color: #ddd"
              :src="form.avatar"
            />
          </v-avatar>
          <v-btn
            style="position: absolute; bottom: 0; right: 0"
            class="mx-2"
            fab
            dark
            dense
            x-small
            color="primary"
            @click="triggerUpload()"
          >
            <v-icon dark> mdi-pencil </v-icon>
          </v-btn>
        </div>
      </div>
      <v-container>
        <v-divider style="padding-bottom: 20px"></v-divider>
        <v-row>
          <v-col
            cols="12"
            md="4"
          >
          <v-text-field
            v-model="form.first_name"
            :rules="[rules.required]"
            label="First name"
          ></v-text-field>
          </v-col>
          <v-col
            cols="12"
            md="4"
          >
          <v-text-field
            v-model="form.middle_name"
            :rules="[rules.required]"
            label="Middle name"
          ></v-text-field>
          </v-col>
          <v-col
            cols="12"
            md="4"
          >
          <v-text-field
            v-model="form.last_name"
            :rules="[rules.required]"
            label="Last name"
          ></v-text-field>
          </v-col>
        </v-row>
          <v-text-field
            v-model="form.position"
            :rules="[rules.required]"
            label="Position"
          ></v-text-field>
          <v-textarea
            v-model="form.intro_experienced"
            :rules="[rules.required]"
            label="Intro Experience"
            rows="3"
            row-height="25"
          ></v-textarea>
          <v-text-field
            v-model="form.mobile"
            :rules="[rules.required]"
            label="Contact Number"
          ></v-text-field>
          <v-text-field
            v-model="form.email"
            :rules="[rules.required, rules.isEmail]"
            label="Email"
          ></v-text-field>
          <v-text-field
            v-model="form.address"
            :rules="[rules.required]"
            label="Address"
          ></v-text-field>
          <v-menu
            ref="menu"
            v-model="dateMenu"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-y
            max-width="290px"
            min-width="auto"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                :rules="[rules.required]"
                v-model="dateFormatted"
                label="Birthdate"
                hint="MM/DD/YYYY format"
                persistent-hint
                v-bind="attrs"
                v-on="on"
                readonly
              ></v-text-field>
            </template>
            <v-date-picker
              v-model="date"
              no-title
              @input="dateMenu = false"
            ></v-date-picker>
          </v-menu>
        </v-container>
    </v-form>
    <v-dialog v-model="showAvatarEditor" width="500">
      <v-card v-if="showAvatarEditor">
        <v-card-title class="text-h5 grey lighten-2"> Avatar </v-card-title>

        <v-card-text>
          <v-form ref="form-avatar" v-model="validAvatar">
            <v-text-field
              v-model="formAvatar.avatar"
              :rules="[rules.required]"
              label="Avatar Link"
            ></v-text-field>
          </v-form>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="validateAvatar()">
            Submit
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import moment from "moment";
export default {
  props: {
    form: Object,
    title: String,
  },

  data() {
    return {
      showAvatarEditor: false,
      formAvatar: {
        avatar: ''
      },
      validAvatar: null,
      valid: true,
      rules: {
        required: (value) => !!value || "Required.",
        isEmail: (email) =>
          !email ||
          /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(email) ||
          "E-mail must be valid",
      },
      date: null,
      dateMenu: false,
    };
  },
  methods: {
    validate() {
      if (this.$refs["form-1"].validate()) {
        console.log("proceed");
        this.$emit("next");
      } else {
        console.log("error");
      }
    },
    validateAvatar() {
      if (this.$refs["form-avatar"].validate()) {
        this.form.avatar = this.formAvatar.avatar
        this.showAvatarEditor = false
      } else {
        console.log("error");
      }
    },
    triggerUpload(){
      // reset file every upload
      const input = this.$refs['upload-image'];
      input.type = 'text';
      input.type = 'file';
      // reset file every upload
      this.$refs['upload-image'].click()
    },
    uploadImage(e){
        let files = e.target.files || e.dataTransfer.files;
        if (!files.length)
          return;

        let results = []
        let reader = new FileReader();
        let i = 0;


        console.log(files[i])
        reader.readAsDataURL(files[i]);

        reader.onload = (e) => {
          results.push(e.target.result)
        };

        reader.onloadend = (a, b) => {
          i++;
          if (i < files.length) {
            console.log(files[i])
            // compressImage(files[i]);
            // reader.readAsDataURL(files[i]);
          } else {
            this.$emit('emitBase64', results)
            this.form.avatar = results[0]
            console.log({results});
          }
        }
      }
  },

  computed: {
    dateFormatted: {
      get() {
        if (!this.form.birthdate) return null;

        return moment(this.form.birthdate).format("LL");
      },
      set(val) {
        this.date = val;
      },
    },
  },

  watch: {
    date: {
      handler(val) {
        if (!val) return null;

        this.form.birthdate = moment(val).format("LL");
      },
      immediate: true,
    },
    showAvatarEditor(v){
      if(!v){
        this.formAvatar = {
          avatar: ''
        }
      }
    }
  },
};
</script>

<style>
</style>