<script setup lang="ts">
import MainLayout from "../../layouts/Main.vue";
import { computed, reactive, ref } from "vue";
import { useI18n } from "vue-i18n";
import Password from "primevue/password";
import InputText from "primevue/inputtext";
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import InputSwitch from "primevue/inputswitch";
import { passwordStrength } from "check-password-strength";

const state = reactive({
  lastError: "",
  email: "",
  password: "",
  name: "",
  savePassword: true,
  emailIsValid: false,
});

const canCreatePassword = computed(() => {
  console.log(
    "canCreatePassword",
    state.emailIsValid,
    !!state.name,
    state.password.length >= 16
  );
  return state.emailIsValid && !!state.name && state.password.length >= 16;
});
const { t } = useI18n(); // use as global scope

const store = useStore();
const router = useRouter();

function checkEmailValidity() {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  state.emailIsValid = emailRegex.test(state.email);
  console.log("email validation", state.email, state.emailIsValid);
}

async function createAccount() {
  try {
    await store.dispatch("wallet/addEmailPasswordAccount", {
      name: state.name,
      savePassword: state.savePassword,
      email: state.email,
      password: state.password,
    });

    router.push({ name: "Accounts" });
  } catch (err: any) {
    const error = err.message ?? err;
    console.error("failed to create account", error, err);
    await store.dispatch("toast/openError", error);
  }
}
</script>
<template>
  <MainLayout>
    <h1>{{ $t("arc76account.title") }}</h1>
    <div v-if="state.lastError">
      <div class="alert alert-danger">
        {{ $t("new_account_pass.last_error") }}: {{ state.lastError }}
      </div>
    </div>
    <p>
      {{ $t("arc76account.description") }}
    </p>
    <p>
      {{ $t("arc76account.description2") }}
    </p>
    <div class="alert alert-danger" v-if="!state.savePassword">
      {{ $t("arc76account.arc_draft") }}
    </div>
    <div class="p-fluid">
      <div class="p-field mb-2">
        <label for="email">{{ $t("arc76account.email") }}</label>
        <InputText
          id="email"
          v-model="state.email"
          @keyup="checkEmailValidity"
        />
      </div>
      <div class="p-field mb-2">
        <label for="w">{{ $t("arc76account.select_password") }}</label>
        <Password
          :input-props="{ autocomplete: 'new-password' }"
          input-id="w"
          v-model="state.password"
          :feedback="false"
          :toggle-mask="true"
        />
      </div>
      <div class="p-field mb-2">
        <label>{{ $t("arc76account.save_password_switch") }}</label>
        <div>
          <InputSwitch class="my-2" v-model="state.savePassword" />
        </div>
      </div>

      <p v-if="!state.savePassword">
        {{ $t("arc76account.password_not_stored") }}
      </p>
      <div class="alert alert-danger" v-if="!state.savePassword">
        {{ $t("arc76account.gui_not_implemented") }}
      </div>
      <div class="p-field mb-2">
        <label for="name">{{ $t("accounts.account_name") }}</label>
        <InputText itemid="name" v-model="state.name" />
      </div>
      <button
        class="btn btn-primary my-2"
        @click="createAccount"
        :disabled="!canCreatePassword"
      >
      {{ $t("newacc.create_account") }}
      </button>
    </div>
  </MainLayout>
</template>
