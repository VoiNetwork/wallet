<template>
  <div
    class="d-flex flex-column align-items-center justify-content-center h-100 m-2"
  >
    <div v-if="newWalletForm" class="card col-12 col-md-6 col-lg-4">
      <div class="card-header">
        {{ $t("login.new_wallet") }}
      </div>
      <div class="card-body">
        <form autocomplete="off" @submit="createWalletClick">
          <label for="newwallet-name">{{ $t("login.new_wallet_name") }}</label>
          <input
            id="newwallet-name"
            v-model="newname"
            class="form-control my-2"
            autocomplete="off"
          />
          <label for="newwallet-pass"
            >{{ $t("login.new_wallet_pass") }}
            <span v-if="strength" :class="strengthClass">{{
              strength
            }}</span></label
          >
          <input
            id="newwallet-pass"
            v-model="pass"
            type="password"
            class="form-control my-2"
            autocomplete="off"
          />
          <input
            type="submit"
            class="btn btn-primary"
            :value="$t('login.new_wallet_button_create')"
          />
          <router-link to="/import-wallet" class="btn btn-light mx-2">
            {{ $t("login.new_wallet_button_import") }}
          </router-link>
          <button
            v-if="wallets.length > 0"
            class="btn btn-light mx-2"
            @click="newWalletForm = false"
          >
            {{ $t("global.go_back") }}
          </button>
          <p class="my-2">
            {{ $t("login.new_wallet_help1") }}
          </p>
          <p class="my-2">
            {{ $t("login.new_wallet_help2") }}
          </p>
        </form>
      </div>
    </div>
    <div v-if="!newWalletForm" class="card col-12 col-md-6 col-lg-4">
      <div class="card-header">
        {{ $t("login.open_wallet") }}
      </div>
      <div class="card-body">
        <form @submit="auth">
          <label for="wallet-select">{{ $t("login.select_wallet") }}</label>
          <select id="wallet-select" v-model="wallet" class="form-control my-2">
            <option v-for="option in wallets" :key="option">
              {{ option }}
            </option>
          </select>
          <label for="wallet-pass">{{ $t("login.wallet_password") }}</label>
          <input
            id="wallet-pass"
            v-model="pass"
            type="password"
            class="form-control my-2"
            autocomplete="on"
          />
          <input
            type="submit"
            class="btn btn-primary"
            :value="$t('login.new_wallet_button_open')"
          />
          <button
            class="btn btn-light mx-2"
            @click="
              newWalletForm = true;
              newname = '';
              pass = '';
            "
          >
            {{ $t("login.new_wallet") }}
          </button>
        </form>
      </div>
    </div>
    <div class="my-5">
      <a
        v-for="lang in $store.state.config.languages"
        :key="lang"
        class="m-2 d-inline-block"
        role="button"
        @click="setLanguage(lang)"
      >
        <img
          :src="'/flags/3x2/' + lang + '.svg'"
          height="50"
          class="border border-3 rounded rounded-3"
        />
      </a>
      <a
        class="m-2"
        href="https://github.com/scholtz/wallet/wiki/I-want-to-help-to-translate"
        target="_blank"
        role="button"
        style="width: 75; height: 50"
        ><svg
          xmlns="http://www.w3.org/2000/svg"
          width="50"
          height="50"
          fill="currentColor"
          class="bi bi-plus-circle"
          viewBox="0 0 16 16"
        >
          <path
            d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z"
          />
          <path
            d="M8 4a.5.5 0 0 1 .5.5v3h3a.5.5 0 0 1 0 1h-3v3a.5.5 0 0 1-1 0v-3h-3a.5.5 0 0 1 0-1h3v-3A.5.5 0 0 1 8 4z"
          /></svg
      ></a>
    </div>
    <div class="my-5">
      <a
        href="https://youtu.be/cCsx7t68DS4"
        target="youtube"
        class="m-2 btn btn-xs btn-light btn-link"
        role="button"
      >
        Youtube - {{ $t("login.basic_usage") }}
      </a>
      <a
        href="https://youtu.be/M0KZvp7AJQs"
        target="youtube"
        class="m-2 btn btn-xs btn-light btn-link"
        role="button"
      >
        Youtube - {{ $t("login.tether_usage") }}
      </a>
      <a
        href="https://github.com/scholtz/wallet/"
        target="github"
        class="m-2 btn btn-xs btn-light btn-link"
        role="button"
      >
        GitHub: {{ $t("login.source_code") }}
      </a>
      <a href="/donate" class="m-2 btn btn-xs btn-light btn-link" role="button">
        Support us
      </a>
    </div>
  </div>
</template>
<script>
import { mapActions } from "vuex";
import { passwordStrength } from "check-password-strength";

export default {
  data() {
    return {
      newname: "",
      newWalletForm: false,
      pass: "",
      wallet: "",
      wallets: [],
    };
  },
  computed: {
    strengthClass() {
      if (!this.pass) return "";
      const ret = passwordStrength(this.pass);
      if (ret.id <= 0) return "badge bg-danger";
      if (ret.id <= 1) return "badge bg-warning text-dark";
      return "badge bg-success";
    },
    strength() {
      if (!this.pass) return "";
      const ret = passwordStrength(this.pass);
      console.log("ret", ret);
      return this.$t("login.strength") + ": " + ret.value;
    },
  },
  async mounted() {
    this.wallets = await this.getWallets();
    this.wallet = localStorage.getItem("lastUsedWallet");

    this.newWalletForm = this.wallets.length == 0;
  },
  methods: {
    ...mapActions({
      getWallets: "wallet/getWallets",
      createWallet: "wallet/createWallet",
      openWallet: "wallet/openWallet",
    }),
    async auth(e) {
      e.preventDefault();
      localStorage.setItem("lastUsedWallet", this.wallet);
      await this.openWallet({ name: this.wallet, pass: this.pass });
      if (this.$store.state.config.noredirect) return;
      if (
        this.$store.state.wallet.lastActiveAccount &&
        this.$store.state.wallet.lastActiveAccountName
      ) {
        const redirectTo =
          "/account/" + this.$store.state.wallet.lastActiveAccount;
        this.$router.push(redirectTo);
        console.log("to", redirectTo);
      } else {
        this.$router.push("/accounts");
      }
    },
    async createWalletClick(e) {
      e.preventDefault();
      const created = await this.createWallet({
        name: this.newname,
        pass: this.pass,
      });
      if (created) {
        if (
          this.$store.state.wallet.lastActiveAccount &&
          this.$store.state.wallet.lastActiveAccountName
        ) {
          this.$router.push(
            "/account/" + this.$store.state.wallet.lastActiveAccount
          );
        } else {
          this.$router.push("/accounts");
        }
      }
    },
    setLanguage(lang) {
      this.$i18n.locale = lang;
      localStorage.setItem("lang", this.$i18n.locale);
    },
  },
};
</script>
