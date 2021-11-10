<template>
  <v-form v-model="formIsValid" ref="form" lazy-validation>
    <v-container>
      <h3>Личные данные</h3>
      <v-container>
        <v-row class="mt-2">
          <v-text-field
            class="mr-5"
            v-model="inputData.surName"
            label="Фамилия"
            :rules="[rules.required, rules.cyrillicSymbols]"
            outlined
            validate-on-blur
          ></v-text-field
          ><v-text-field
            class="mr-5"
            v-model="inputData.name"
            label="Имя"
            required
            :rules="[rules.required, rules.cyrillicSymbols]"
            outlined
            validate-on-blur
          ></v-text-field
          ><v-text-field
            v-model="inputData.middleName"
            label="Отчество"
            required
            :rules="[rules.required, rules.cyrillicSymbols]"
            outlined
            validate-on-blur
          ></v-text-field
        ></v-row>

        <v-row>
          <v-text-field
            class="mr-5"
            style="max-width: 240px"
            v-model="inputData.birthday"
            label="Дата рождения"
            v-mask="'##.##.####'"
            placeholder="дд.мм.гггг"
            :rules="[rules.notEarlierThanToday]"
            validate-on-blur
            outlined
          ></v-text-field>
        </v-row>
        <v-row
          ><v-text-field
            class="mr-5"
            style="max-width: 480px"
            v-model="inputData.email"
            :rules="[rules.email, rules.required]"
            label="E-mail"
            outlined
            validate-on-blur
          ></v-text-field
        ></v-row>
      </v-container>
      <v-row>
        <v-container>
          <span>Выберите пол</span>
          <v-radio-group v-model="inputData.sex" row>
            <v-radio label="Мужской" value="male"></v-radio>
            <v-radio label="Женский" value="female"></v-radio> </v-radio-group
        ></v-container>
      </v-row>
      <h3>Паспортные данные</h3>

      <v-container class="pl-0"
        ><v-overflow-btn
          editable
          id="country-dropdown"
          v-model="inputData.choosenCountryID"
          :items="countryData"
          item-value="id"
          item-text="nationality"
          label="Гражданство"
          outlined
          style="max-width: 350px"
        />
      </v-container>
      <v-container v-if="inputData.choosenCountryID === 8604">
        <v-row>
          <v-text-field
            class="mr-5"
            v-model="inputData.passportSeries"
            label="Серия паспорта"
            :rules="[rules.passportSeriesRu]"
            validate-on-blur
            outlined
          ></v-text-field
          ><v-text-field
            class="mr-5"
            v-model="inputData.passportNumber"
            label="Номер паспорта"
            required
            :rules="[rules.passportNumberRu]"
            validate-on-blur
            outlined
          ></v-text-field
          ><v-text-field
            v-model="inputData.dateOfIssue"
            label="Дата выдачи"
            required
            v-mask="'##.##.####'"
            placeholder="дд.мм.гггг"
            :rules="[rules.notEarlierThanToday]"
            validate-on-blur
            outlined
          ></v-text-field
        ></v-row> </v-container
      ><v-container v-else-if="inputData.choosenCountryID">
        <v-row
          ><v-text-field
            class="mr-5"
            v-model="inputData.latinSurname"
            label="Фамилия на латинице"
            hint="Иностранцы заполняют латинскими буквами. Например, Ivan Ivanov"
            :rules="[rules.latinSymbols]"
            validate-on-blur
            persistent-hint
            outlined
          ></v-text-field
          ><v-text-field
            class="mr-5"
            v-model="inputData.latinName"
            label="Имя на латинице"
            :rules="[rules.latinSymbols]"
            validate-on-blur
            outlined
          ></v-text-field
        ></v-row>
        <v-row>
          <v-text-field
            class="mr-5"
            v-model="inputData.passportNumber"
            label="Номер паспорта"
            :rules="[rules.onlyDigits]"
            validate-on-blur
            outlined
          ></v-text-field
          ><v-overflow-btn
            class="mr-5"
            style="width: 120px"
            editable
            v-model="inputData.passportType"
            :items="countryData"
            item-value="id"
            item-text="nationality"
            label="Страна выдачи"
            outlined
          />
          <v-overflow-btn
            editable
            style="width: 120px"
            v-model="inputData.countryOfIssue"
            :items="passportTypes"
            item-value="id"
            item-text="type"
            label="Тип паспорта"
            outlined
          />
        </v-row>
      </v-container>
      <v-container v-if="inputData.choosenCountryID">
        <span>Меняли ли фамилию или имя?</span>
        <v-radio-group @change="handleNameChangedState" row>
          <v-radio label="Нет" value="No"></v-radio>
          <v-radio label="Да" value="Yes"></v-radio>
        </v-radio-group>
      </v-container>
      <v-container v-if="inputData.nameChanged">
        <v-row>
          <v-text-field
            class="mr-5"
            v-model="inputData.previousSurname"
            label="Предыдущая Фамилия"
            :rules="[rules.cyrillicSymbols]"
            validate-on-blur
            outlined
          ></v-text-field
          ><v-text-field
            class="mr-5"
            v-model="inputData.previousName"
            label="Предыдущее Имя"
            :rules="[rules.cyrillicSymbols]"
            validate-on-blur
            outlined
          ></v-text-field
        ></v-row>
      </v-container>
      <v-container class="">
        <v-btn @click="handleSubmitClick">SUBMIT</v-btn></v-container
      >
    </v-container>
  </v-form>
</template>

<script>
import countryData from "@/assets/data/citizenships.json";
import passportTypes from "@/assets/data/passport-types.json";
import { parse } from "date-fns";

export default {
  data() {
    return {
      formIsValid: false,
      countryData: countryData,
      passportTypes: passportTypes,
      inputData: {
        surName: "",
        name: "",
        middleName: "",
        birthday: null,
        sex: "",
        email: "",
        choosenCountryID: null,
        passportSeries: null,
        passportNumber: null,
        dateOfIssue: "",
        latinName: "",
        latinSurname: "",
        countryOfIssue: "",
        passportType: "",
        nameChanged: false,
        previousName: "",
        previousSurname: "",
      },
      form: false,
      rules: {
        email: (v) => !!(v || "").match(/@/) || "Введите корректный email",
        required: (v) => !!v || "Поле обязательно для заполнения",
        cyrillicSymbols: (v) =>
          !!(v || "").match("^[А-Яа-я]+$") || "Только русские символы",
        notEarlierThanToday: (v) =>
          parse(v, "dd.mm.yyyy", new Date()) < new Date() ||
          "Введите корректную дату",
        latinSymbols: (v) =>
          !!(v || "").match("^[A-Za-z]+$") || "Только латинские символы",
        onlyLetters: (v) =>
          !!(v || "").match("^[a-zA-Zа-яА-Я]+$") || "Только буквы",
        passportSeriesRu: (v) =>
          !!(v || "").match("^([0-9]{4})$") ||
          "Серия паспорта должна состоять из 4 цифр",
        passportNumberRu: (v) =>
          !!(v || "").match("^([0-9]{6})$") ||
          "Номер паспорта должен состоять из 6 цифр",
        onlyDigits: (v) =>
          !!(v || "").match("^[0-9]+$") ||
          "Номер паспорта может состоять только из цифр",
      },
    };
  },
  methods: {
    handleNameChangedState(newValue) {
      this.$data.inputData.nameChanged = newValue === "Yes";
    },
    handleSubmitClick() {
      this.$refs.form.validate();
      if (this.$data.formIsValid)
        console.log(JSON.stringify(this.$data.inputData));
    },
  },
};
</script>
