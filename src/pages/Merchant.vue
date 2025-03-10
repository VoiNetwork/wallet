<template>
  <PublicLayout>
    <template #header>
      <span />
    </template>
    <template #footer>
      <span />
    </template>
    <div class="container">
      <h1>
        {{ $t("merchant.make_payment") }}
        <span v-if="asset">{{ asset.name }}</span>
      </h1>
      <div v-if="b64decode && !b64decode.error" class="my-3">
        {{ $t("merchant.pay") }} {{ b64decode.payamount }}
        <span v-if="asset">{{ asset["unit-name"] }}</span>
        {{ $t("merchant.to_address") }} {{ b64decode.payTo }}
        {{ $t("merchant.please") }}
        <table class="w-100">
          <tr v-if="b64decode.network">
            <td>{{ $t("merchant.network") }}:</td>
            <td>
              <code>{{ b64decode.network }}</code>
            </td>
          </tr>
          <tr v-if="b64decode.paynote">
            <td>{{ $t("merchant.matching_symbol") }}:</td>
            <td>
              <code>{{ b64decode.paynote }}</code>
            </td>
          </tr>
          <tr v-if="b64decode.fee">
            <td>{{ $t("merchant.network_fee") }}:</td>
            <td>
              <code>{{ b64decode.fee }} ALGO</code>
            </td>
          </tr>
        </table>
      </div>
      <div v-else-if="b64decode.error" class="my-3">
        {{ b64decode.error }}
      </div>
      <div v-else>
        <span
          class="spinner-grow spinner-grow-sm"
          role="status"
          aria-hidden="true"
        />
      </div>
      <div v-if="!isPaid">
        <a
          :href="origcode"
          :title="origcode"
          role="button"
          class="btn btn-primary btn-xl mb-3"
          >{{ $t("merchant.pay_qr") }}</a
        >
        <a
          :href="origcode"
          :title="origcode"
          class="btn btn-light btn-xl mx-3 mb-3"
          >{{ $t("merchant.pay_nativewallet") }}</a
        >
        <a
          :href="codeP2"
          :title="codeP2"
          class="btn btn-light btn-xl mx-3 mb-3"
          >{{ $t("merchant.pay_webwallet") }}</a
        >
        <a
          v-if="settings.cancel"
          :href="settings.cancel"
          class="btn btn-light btn-xl mb-3"
          >{{ $t("merchant.cancel_payment") }}</a
        >
      </div>

      <p v-if="isPaid" class="alert alert-success mb-3">
        {{ $t("merchant.payment_received") }}
      </p>

      <form
        v-if="isPaid && settings.success"
        ref="sendToMerchant"
        :action="settings.success"
        method="POST"
      >
        <input
          type="hidden"
          name="txId"
          :value="transactions.transactions[0].id"
        />
        <input
          type="submit"
          class="btn btn-primary btn-xl my-2"
          :value="$t('merchant.go_back_to_merchant')"
        />
        <span class="badge bg-info text-dark m-2">({{ countDown }})</span>
      </form>

      <p v-if="!isPaid" class="alert alert-primary mb-3">
        <span
          class="spinner-grow spinner-grow-sm"
          role="status"
          aria-hidden="true"
        />
        {{ $t("merchant.waiting_for_payment") }}
      </p>
      <div class="row">
        <div class="col-12 col-lg-6">
          <QRCodeVue3
            v-if="!isPaid"
            :title="origcode"
            :width="400"
            :height="400"
            :value="origcode"
            :key="origcode"
            :qr-options="{ errorCorrectionLevel: 'H' }"
            :image="assetImage"
          />
        </div>
        <div class="col-12 col-lg-6">
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
        </div>
      </div>
    </div>
  </PublicLayout>
</template>

<script>
import PublicLayout from "../layouts/Public.vue";
import QRCodeVue3 from "qrcode-vue3";
import { mapActions } from "vuex";
import aprotocol from "../shared/algorand-protocol-parse";
import { v4 as uuidv4 } from "uuid";
import base64url from "base64url";

export default {
  components: {
    PublicLayout,
    QRCodeVue3,
  },
  data() {
    return {
      origcode: "",
      codeP2: "",
      b64decode: {},
      settings: {},
      asset: {},

      transactions: [],
      formSent: false,
      countDown: null,
    };
  },
  computed: {
    assetImage() {
      if (this.b64decode.asset == 312769) {
        return "/img/tether-usdt-logo-96.png";
      }
      if (this.asset["asset-id"] === "") {
        return "/img/algorand-algo-logo-96.png";
      }
      return "";
    },
    isPaid() {
      return (
        this.transactions &&
        this.transactions.transactions &&
        this.transactions.transactions.length > 0
      );
    },
  },
  mounted() {
    console.log("qrcode", this.qrcode, this.account);
    try {
      try {
        this.origcode = atob(this.$route.params.b64);
      } catch {
        this.origcode = base64url.decode(this.$route.params.b64);
      }
      if (this.$route.params.settings) {
        let settings = "";
        try {
          settings = atob(this.$route.params.settings);
        } catch {
          settings = base64url.decode(this.$route.params.settings);
        }
        this.settings = JSON.parse(settings);
      }
    } catch (e) {
      this.openError(e.message);
    }
    this.b64decode = aprotocol.parseAlgorandProtocolParameters(this.origcode);
    this.setNetwork();
    this.makeAsset();
    if (!this.b64decode.paynote) {
      this.b64decode.paynote = "a-wallet/" + uuidv4();
      this.origcode = this.origcode + "&xnote=" + this.b64decode.paynote;
    }
    console.log("this.b64decode ", this.b64decode, this.asset, this.settings);
    window.setInterval(() => {
      this.getNotifications();
    }, 5000);
    this.setWebProtocol();
  },
  methods: {
    ...mapActions({
      searchForTransactions: "indexer/searchForTransactions",
      getAsset: "indexer/getAsset",
      openError: "toast/openError",
      setHosts: "config/setHosts",
    }),
    setWebProtocol() {
      if (this.origcode.startsWith("algorand:")) {
        this.codeP2 = "web+" + this.origcode;
      } else {
        this.codeP2 = "web+algorand://" + this.origcode;
      }
    },
    async lowerCountDown() {
      if (this.countDown > 0) {
        this.countDown = this.countDown - 1;
      } else {
        if (!this.formSent) {
          this.formSent = true;
          this.$refs.sendToMerchant.submit();
        }
      }
    },
    async getNotifications() {
      this.transactions = await this.searchForTransactions({
        addr: this.b64decode.payTo,
        note: this.b64decode.paynote,
      });
      if (this.isPaid) {
        if (this.settings.success) {
          if (this.countDown === null) {
            this.countDown = 10;
            window.setInterval(() => {
              this.lowerCountDown();
            }, 1000);
          }
        }
      }
      console.log("transactions", this.transactions);
    },
    async makeAsset() {
      this.asset = {};
      if (this.b64decode && this.b64decode.asset > 0) {
        const newAsset = await this.getAsset({
          assetIndex: this.b64decode.asset,
        });
        if (newAsset && newAsset.decimals !== undefined) {
          this.asset = newAsset;
        } else {
          const errorMsg =
            this.$t("merchant.error_asset") + this.b64decode.asset;
          this.openError(errorMsg);
          this.b64decode.error = errorMsg;
          return;
        }
      } else {
        this.asset = {
          "asset-id": "",
          amount: 0,
          name: "ALG",
          decimals: 6,
          "unit-name": "Algo",
        };
      }
      // transfer to asset decimals
      this.b64decode.payamountLong = this.b64decode.payamount;
      this.b64decode.payamount =
        this.b64decode.payamount / Math.pow(10, this.asset.decimals);

      if (!this.b64decode.feeLong) {
        if (!this.b64decode.fee) {
          this.b64decode.fee = 0.001;
          this.b64decode.feeLong = 1000;
        } else {
          this.b64decode.feeLong = this.b64decode.fee;
          this.b64decode.fee = this.b64decode.fee / 1000000;
        }
      }
    },
    setNetwork() {
      console.log("this.b64decode.network", this.b64decode.network);
      if (
        this.b64decode.network == "mainnet" ||
        this.b64decode.network == "mainnet-v1.0"
      ) {
        this.setHosts({
          algod: "https://mainnet-api.algonode.cloud",
          kmd: "https://kmd.h2.a-wallet.net",
          indexer: "https://mainnet-idx.algonode.cloud",
        });
      }
      if (this.b64decode.network == "aramidmain") {
        this.setHosts({
          algod: "https://algod.aramidmain.a-wallet.net",
          kmd: "?",
          indexer: "https://indexer.aramidmain.a-wallet.net",
        });
      }
      if (
        this.b64decode.network == "testnet" ||
        this.b64decode.network == "testnet-v1.0"
      ) {
        console.log("setting testnet");
        this.setHosts({
          algod: "https://testnet-api.algonode.cloud",
          kmd: "?",
          indexer: "https://testnet-idx.algonode.cloud",
        });
      }
      if (this.b64decode.network == "sandbox") {
        console.log("setting sandbox");
        this.setHosts({
          algod: "http://localhost:4001",
          kmd: "http://localhost:4002",
          indexer: "http://localhost:8980",
          algodToken:
            "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          kmdToken:
            "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          indexerToken:
            "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        });
      }
    },
    setLanguage(lang) {
      this.$i18n.locale = lang;
      localStorage.setItem("lang", this.$i18n.locale);
    },
  },
};
</script>
