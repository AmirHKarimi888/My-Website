<script>
import { ref } from "vue";
import { useField, useForm } from "vee-validate";
import { url } from "../api";
import { Action } from "../httpService";

//localStorage.setItem("authedUser", `{ "username": "", "email": "", "avatar": "", "isAdmin": ${ false }, "isLogin": ${ false } }`);

export default {
  setup() {
    const { handleSubmit, handleReset } = useForm({
      validationSchema: {
        email(value) {
          if (/^[a-z.0-9.]+@[a-z.-]+\.[a-z]+$/i.test(value)) return true;
          return "Must be a valid e-mail.";
        },
        password(value) {
          if (value?.length > 9 && /[0-9-]+/.test(value)) return true;

          return "Password needs to be at least 9 digits.";
        },
        checkbox(value) {
          if (value === "1") return true;

          return "Must be checked.";
        },
      },
    });
    const email = useField("email");
    const password = useField("password");
    const checkbox = useField("checkbox");

    const users = ref([]);
    Action.get(
      url + "/users",
      (response) => (users.value = response.data)
    ).then(() => {});

    //   (values) => {
    //   //alert(JSON.stringify(values, null, 2))
    // }

    const notMatch = ref("");

    const submit = handleSubmit(() => {
      users.value.map((user) => {
        if (
          user.email === email.value.value &&
          user.password === password.value.value
        ) {
          notMatch.value = "";
          localStorage.setItem("authedUser", user.id);
          setTimeout(() => {
            window.location.href = "/";
          }, 1000);
        } else {
          notMatch.value = "Email and password don't match.";
        }
      });
    });

    return {
      password,
      email,
      checkbox,
      notMatch,
      submit,
      handleReset,
    };
  },
};
</script>

<template>
  <div class="login">
    <h1>Login</h1>
    <v-sheet width="300" class="mx-auto mt-15">
      <form @submit.prevent="submit">
        <v-text-field
          v-model="email.value.value"
          :error-messages="email.errorMessage.value"
          label="E-mail"
        ></v-text-field>

        <v-text-field
          v-model="password.value.value"
          :counter="7"
          :error-messages="password.errorMessage.value"
          label="Password"
        ></v-text-field>

        <v-checkbox
          v-model="checkbox.value.value"
          :error-messages="checkbox.errorMessage.value"
          value="1"
          label="I am not a bot"
          type="checkbox"
        ></v-checkbox>

        <v-btn class="me-4" type="submit"> Login </v-btn>

        <v-btn @click="handleReset"> Clear Form </v-btn>

        <p class="mt-3 text-red">{{ notMatch }}</p>
      </form>
    </v-sheet>
  </div>
</template>
