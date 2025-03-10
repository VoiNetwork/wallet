<template>
  <MainLayout>
    <h1>
      {{ $t("acc_overview.title") }} -
      {{ $store.state.wallet.lastActiveAccountName }}
    </h1>
    <p>
      <button
        class="btn btn-light btn-xs m-2 float-end"
        @click="displayDeleteDialog = true"
      >
        <div>{{ $t("acc_overview.delete") }}</div>
      </button>

      <button
        v-if="account"
        class="btn btn-light btn-xs m-2 float-end"
        @click="hideAccountClick"
      >
        <div v-if="account.isHidden">Unhide account</div>
        <div v-else>Hide account</div>
      </button>

      <Dialog
        v-model:visible="displayDeleteDialog"
        :header="$t('acc_overview.delete_header')"
        :modal="true"
      >
        <p>{{ $t("acc_overview.delete_confirm") }}</p>
        <p v-if="account">
          <b>{{ account.name }}</b>
        </p>
        <p v-if="account">
          {{ account.addr }}
        </p>

        <template #footer>
          <button
            class="btn btn-xs btn-primary"
            @click="displayDeleteDialog = false"
          >
            {{ $t("global.cancel") }}</button
          ><button class="btn btn-xs btn-danger" @click="deleteAccountClick">
            {{ $t("acc_overview.delete_confirm_button") }}
          </button>
        </template>
      </Dialog>
      <Dialog
        v-model:visible="displayOnlineOfflineDialog"
        :header="$t('onlineofflinedialog.header')"
        :modal="true"
      >
        <p>{{ $t("onlineofflinedialog.warning") }}</p>
        <input
          v-model="onlineRounds"
          class="form-control"
          type="number"
          min="0"
          max="2000000"
          step="10000"
        />
        <p>
          {{ $t("onlineofflinedialog.host") }}:
          {{ $store.state.config.participation }}
        </p>
        <template #footer>
          <button
            class="btn btn-xs btn-light"
            @click="displayOnlineOfflineDialog = false"
          >
            {{ $t("global.cancel") }}</button
          ><button
            class="btn btn-xs btn-primary"
            @click="setAccountOnlineAtParticipationNode"
          >
            {{ $t("onlineofflinedialog.makeOnline") }}</button
          ><button
            v-if="account['status'] == 'Online'"
            class="btn btn-xs btn-danger"
            @click="setAccountOfflineAtParticipationNode"
          >
            {{ $t("onlineofflinedialog.makeOffline") }}
          </button>
        </template>
      </Dialog>
      <router-link
        v-if="canSign"
        :to="'/accounts/pay/' + $route.params.account"
        class="btn btn-light btn-xs me-2 my-2"
      >
        {{ $t("acc_overview.pay") }}
      </router-link>
      <router-link
        v-if="canSign"
        :to="'/accounts/rekey/' + $route.params.account"
        class="btn btn-light btn-xs me-2 my-2"
      >
        {{ $t("acc_overview.rekey") }}
      </router-link>
      <router-link
        v-if="canSign"
        :to="'/account/optin/' + $route.params.account"
        class="btn btn-light btn-xs me-2 my-2"
      >
        {{ $t("acc_overview.asset_optin") }}
      </router-link>
      <router-link
        :to="'/receive-payment/' + $route.params.account"
        class="btn btn-light btn-xs me-2 my-2"
      >
        {{ $t("acc_overview.receive_payment") }}
      </router-link>
      <router-link
        :to="'/payment-gateway/' + $route.params.account"
        class="btn btn-light btn-xs me-2 my-2"
      >
        {{ $t("acc_overview.payment_gateway") }}
      </router-link>
      <router-link
        v-if="canSign"
        :to="'/account/connect/' + $route.params.account"
        class="btn btn-light btn-xs me-2 my-2"
      >
        {{ $t("acc_overview.connect") }}
      </router-link>
      <router-link
        v-if="account && (account.sk || account.params)"
        :to="'/vote/ask/'"
        class="btn btn-light btn-xs me-2 my-2"
      >
        {{ $t("acc_overview.ask_question") }}
      </router-link>

      <router-link
        v-if="$store.state.config.dev"
        :to="'/arc14/' + $route.params.account"
        class="btn btn-light btn-xs me-2 my-2"
      >
        ARC14
      </router-link>
    </p>
    <div
      v-if="account && account.network != this.$store.state.config.env"
      class="alert alert-danger"
    >
      <div v-if="account.network">
        This account is assigned to network {{ account.network }}.
      </div>
      <div v-else>This account is not assigned to any network.</div>
      <button class="btn btn-primary my-2" @click="assignToCurrentNetwork">
        Assign account to {{ this.$store.state.config.env }}
      </button>
    </div>
    <table v-if="account" class="w-100">
      <tr>
        <th>{{ $t("acc_overview.name") }}:</th>
        <td>{{ account["name"] }}</td>
        <td rowspan="15" class="text-end">
          <QRCodeVue3
            class="d-md-none d-lg-block"
            :width="400"
            :height="400"
            :value="account.addr"
            :qr-options="{ errorCorrectionLevel: 'H' }"
            image="/img/algorand-algo-logo-96.png"
            :image-options="{
              hideBackgroundDots: true,
              imageSize: 0.4,
              margin: 10,
            }"
          />
        </td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.type") }}:</th>
        <td>
          <div v-if="account.rekeyedTo" class="badge bg-danger">
            {{ $t("acc_type.rekeyed") }}
          </div>
          <div v-else-if="account.sk" class="badge bg-primary">
            {{ $t("acc_type.basic_account") }}
          </div>
          <div
            v-else-if="account.type == '2fa'"
            class="badge bg-primary text-light"
          >
            2FA Multisig
          </div>
          <div
            v-else-if="account.type == '2faApi'"
            class="badge bg-light text-dark"
          >
            2FA API technical account
          </div>
          <div v-else-if="account.params" class="badge bg-warning text-dark">
            {{ $t("acc_type.multisig_account") }}
          </div>
          <div
            v-else-if="account.type == 'ledger'"
            class="badge bg-success text-light"
          >
            {{ $t("acc_type.ledger_account") }}
          </div>
          <div
            v-else-if="account.type == 'wc'"
            class="badge bg-success text-light"
          >
            {{ $t("acc_type.wc_account") }}
          </div>
          <div v-else class="badge bg-info text-dark">
            {{ $t("acc_type.public_account") }}
          </div>
        </td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.address") }}:</th>
        <td>
          <button
            class="btn btn-xs btn-light m-1"
            :title="$t('global.copy_address')"
            @click="copyToClipboard(account.addr)"
          >
            <i class="pi pi-copy" />
          </button>
          {{ account.addr }}
        </td>
      </tr>
      <tr v-if="account.rekeyedTo">
        <th>{{ $t("acc_overview.rekeyedTo") }}:</th>
        <td>
          {{ account.rekeyedTo }}

          <div v-if="rekeyedToInfo">
            <div v-if="rekeyedToInfo.rekeyedTo" class="badge bg-danger">
              {{ $t("acc_type.rekeyed") }}
            </div>
            <div v-else-if="rekeyedToInfo.sk" class="badge bg-primary">
              {{ $t("acc_type.basic_account") }}
            </div>
            <div
              v-else-if="rekeyedToInfo.type == '2fa'"
              class="badge bg-primary text-light"
            >
              2FA Multisig
            </div>
            <div
              v-else-if="rekeyedToInfo.type == '2faApi'"
              class="badge bg-light text-dark"
            >
              2FA API technical account
            </div>
            <div
              v-else-if="rekeyedToInfo.params"
              class="badge bg-warning text-dark"
            >
              {{ $t("acc_type.multisig_account") }}
            </div>
            <div
              v-else-if="rekeyedToInfo.type == 'ledger'"
              class="badge bg-success text-light"
            >
              {{ $t("acc_type.ledger_account") }}
            </div>
            <div
              v-else-if="rekeyedToInfo.type == 'wc'"
              class="badge bg-success text-light"
            >
              {{ $t("acc_type.wc_account") }}
            </div>
            <div v-else class="badge bg-info text-dark">
              {{ $t("acc_type.public_account") }}
            </div>
            <table v-if="rekeyedToInfo.params">
              <tr v-if="rekeyedToInfo.params">
                <th>{{ $t("acc_overview.multisignature_threshold") }}:</th>
                <td>{{ rekeyedToInfo.params.threshold }}</td>
              </tr>
              <tr v-if="rekeyedToInfo.params">
                <th>{{ $t("acc_overview.multisignature_addresses") }}:</th>
                <td>{{ rekeyedToInfo.params.addrs }}</td>
              </tr>
            </table>
          </div>
        </td>
      </tr>
      <tr v-if="account.type == 'ledger'">
        <th>{{ $t("acc_overview.account0") }}:</th>
        <td>{{ account.addr0 }}</td>
      </tr>
      <tr v-if="account.type == 'ledger'">
        <th>{{ $t("acc_overview.slot") }}:</th>
        <td>{{ account.slot }}</td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.amount") }}:</th>
        <td>{{ $filters.formatCurrency(account.amount) }}</td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.amount_without_pending") }}:</th>
        <td>
          {{
            $filters.formatCurrency(account["amount-without-pending-rewards"])
          }}
        </td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.rewards") }}:</th>
        <td>{{ $filters.formatCurrency(account["rewards"]) }}</td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.pending_rewards") }}:</th>
        <td>{{ $filters.formatCurrency(account["pending-rewards"]) }}</td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.reward_base") }}:</th>
        <td>{{ account["reward-base"] }}</td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.round") }}:</th>
        <td>{{ account["round"] }}</td>
      </tr>
      <tr>
        <th>{{ $t("acc_overview.status") }}:</th>
        <td v-if="changeOnline">
          <span
            class="spinner-grow spinner-grow-sm"
            role="status"
            aria-hidden="true"
          />
          Setting your account to online state. Please wait a while
        </td>
        <td v-else-if="changeOffline">
          <span
            class="spinner-grow spinner-grow-sm"
            role="status"
            aria-hidden="true"
          />
          Setting your account to offline state. Please wait a while
        </td>
        <td v-else-if="$store.state.config.participation">
          <button
            class="btn btn-light btn-xs"
            @click="displayOnlineOfflineDialog = true"
          >
            {{ account["status"] ?? "?" }}
          </button>
        </td>
        <td v-else>
          {{ account["status"] ?? "?" }}
        </td>
      </tr>
      <tr
        v-if="
          $store &&
          $store.state &&
          $store.state.config &&
          $store.state.config.dev
        "
      >
        <th>{{ $t("acc_overview.apps_local_state") }}:</th>
        <td>{{ account["apps-local-state"] }}</td>
      </tr>
      <tr
        v-if="
          $store &&
          $store.state &&
          $store.state.config &&
          $store.state.config.dev
        "
      >
        <th>{{ $t("acc_overview.apps_total_schema") }}:</th>
        <td>{{ account["apps-total-schema"] }}</td>
      </tr>
      <tr v-if="account && account['assets'] && account['assets'].length > 0">
        <th>{{ $t("acc_overview.assets") }}:</th>
        <td>
          <table class="w-100">
            <thead>
              <tr>
                <th>ASA Amount</th>
                <th>ASA ID</th>
                <th>IsFrozen</th>
                <th>Creator</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="asset in account['assets']" :key="asset['asset-id']">
                <td>
                  {{
                    $filters.formatCurrency(
                      asset["amount"],
                      getAssetName(asset["asset-id"]),
                      getAssetDecimals(asset["asset-id"])
                    )
                  }}
                </td>
                <td>{{ asset["asset-id"] }}</td>
                <td>{{ asset["is-frozen"] }}</td>
                <td>{{ asset["creator"] }}</td>
              </tr>
            </tbody>
          </table>
        </td>
      </tr>
      <tr
        v-if="
          $store &&
          $store.state &&
          $store.state.config &&
          $store.state.config.dev
        "
      >
        <th>{{ $t("acc_overview.created_apps") }}:</th>
        <td>{{ account["created-apps"] }}</td>
      </tr>
      <tr v-if="account.params">
        <th>{{ $t("acc_overview.multisignature_threshold") }}:</th>
        <td>{{ account.params.threshold }}</td>
      </tr>
      <tr v-if="account.params">
        <th>{{ $t("acc_overview.multisignature_addresses") }}:</th>
        <td>{{ account.params.addrs }}</td>
      </tr>
      <tr>
        <th />
        <td>
          <button class="btn btn-light btn-xs" @click="reloadAccount">
            {{ $t("acc_overview.refresh") }}
          </button>
        </td>
      </tr>
    </table>

    <h2>{{ $t("acc_overview.transactions") }}</h2>

    <DataTable
      v-model:selection="selection"
      :value="transactions"
      responsive-layout="scroll"
      selection-mode="single"
      :paginator="true"
      :rows="20"
    >
      <template #empty>
        {{ $t("acc_overview.no_transactions") }}
      </template>
      <Column
        field="tx-type"
        :header="$t('acc_overview.type')"
        :sortable="true"
      />
      <Column
        field="round-time"
        :header="$t('acc_overview.time')"
        :sortable="true"
      >
        <template #body="slotProps">
          <div v-if="slotProps.column.props.field in slotProps.data">
            {{
              $filters.formatDateTime(
                slotProps.data[slotProps.column.props.field]
              )
            }}
          </div>
        </template>
      </Column>
      <Column
        field="payment-transaction.amount"
        :header="$t('acc_overview.tr_amount')"
        :sortable="true"
      >
        <template #body="slotProps">
          <div
            v-if="
              slotProps.data['tx-type'] == 'pay' &&
              'payment-transaction' in slotProps.data &&
              'amount' in slotProps.data['payment-transaction']
            "
            class="text-end"
          >
            {{
              $filters.formatCurrency(
                slotProps.data["payment-transaction"]["amount"]
              )
            }}
          </div>
          <div
            v-if="
              slotProps.data['tx-type'] == 'axfer' &&
              'asset-transfer-transaction' in slotProps.data &&
              'amount' in slotProps.data['asset-transfer-transaction']
            "
            class="text-end"
          >
            {{
              $filters.formatCurrency(
                slotProps.data["asset-transfer-transaction"]["amount"],
                getAssetName(
                  slotProps.data["asset-transfer-transaction"]["asset-id"]
                ),

                getAssetDecimals(asset["asset-id"])
              )
            }}
          </div>
        </template>
      </Column>
      <Column
        field="sender"
        :header="$t('acc_overview.sender')"
        :sortable="true"
        style-class="not-show-at-start"
      />
      <Column
        field="payment-transaction.receiver"
        :header="$t('acc_overview.receiver')"
        :sortable="true"
        style-class="not-show-at-start"
      />
      <Column
        field="receiver-rewards"
        :header="$t('acc_overview.receiver_rewards')"
        :sortable="true"
      >
        <template #body="slotProps">
          <div
            v-if="slotProps.column.props.field in slotProps.data"
            class="text-end"
          >
            {{
              $filters.formatCurrency(
                slotProps.data[slotProps.column.props.field]
              )
            }}
          </div>
        </template>
      </Column>
      <Column field="fee" :header="$t('acc_overview.fee')" :sortable="true">
        <template #body="slotProps">
          <div
            v-if="slotProps.column.props.field in slotProps.data"
            class="text-end"
          >
            {{
              $filters.formatCurrency(
                slotProps.data[slotProps.column.props.field]
              )
            }}
          </div>
        </template>
      </Column>
      <Column
        field="confirmed-round"
        :header="$t('acc_overview.confirmed_round')"
        :sortable="true"
      />
    </DataTable>
  </MainLayout>
</template>

<script>
import MainLayout from "../layouts/Main.vue";
import { mapActions } from "vuex";
import { PrimeIcons } from "primevue/api";
import copy from "copy-to-clipboard";

import QRCodeVue3 from "qrcode-vue3";
export default {
  components: {
    MainLayout,
    QRCodeVue3,
  },
  data() {
    return {
      displayDeleteDialog: false,
      displayOnlineOfflineDialog: false,
      transactions: [],
      selection: null,
      assets: [],
      asset: "",
      icons: [PrimeIcons.COPY],
      changeOnline: false,
      onlineRounds: 500000,
    };
  },
  computed: {
    canSign() {
      if (!this.account) return false;

      if (this.account.rekeyedTo) {
        if (!this.rekeyedToInfo) return false;

        return (
          this.rekeyedToInfo.sk ||
          this.rekeyedToInfo.params ||
          this.rekeyedToInfo.type == "ledger" ||
          this.rekeyedToInfo.type == "wc"
        );
      }

      return (
        this.account.sk ||
        this.account.params ||
        this.account.type == "ledger" ||
        this.account.type == "wc"
      );
    },
    account() {
      var acc = this.$store.state.wallet.privateAccounts.find(
        (a) =>
          a.addr == this.$route.params.account &&
          (a.network === undefined || a.network == this.$store.state.config.env)
      );
      if (acc) return acc;
      return this.$store.state.wallet.privateAccounts.find(
        (a) => a.addr == this.$route.params.account
      );
    },
    lastActiveAccountAddr() {
      return this.$store.state.wallet.lastActiveAccount;
    },
    rekeyedToInfo() {
      return this.$store.state.wallet.privateAccounts.find(
        (a) =>
          a.addr == this.account.rekeyedTo &&
          (a.network === undefined || a.network == this.$store.state.config.env)
      );
    },
    rekeyedMultisigParams() {
      const rekeyedInfo = this.$store.state.wallet.privateAccounts.find(
        (a) =>
          a.addr == this.account.rekeyedTo &&
          (a.network === undefined || a.network == this.$store.state.config.env)
      );
      if (!rekeyedInfo) return null;
      return rekeyedInfo.params;
    },
  },
  watch: {
    async selection() {
      await this.setTransaction({ transaction: this.selection });
      console.log("this.selection", this.selection);
      if (this.selection.id) {
        this.$router.push("/transaction/" + this.selection.id);
      }
    },
    account() {
      this.makeAssets();
    },
  },
  async mounted() {
    await this.reloadAccount();
    await this.makeAssets();
    this.prolong();

    if (this.account && !this.account.network) {
      await this.assignToCurrentNetwork();
    }
  },
  methods: {
    ...mapActions({
      accountInformation: "indexer/accountInformation",
      updateAccount: "wallet/updateAccount",
      lastActiveAccount: "wallet/lastActiveAccount",
      deleteAccount: "wallet/deleteAccount",
      searchForTransactions: "indexer/searchForTransactions",
      setTransaction: "wallet/setTransaction",
      getAsset: "indexer/getAsset",
      prolong: "wallet/prolong",
      setAccountOnline: "kmd/setAccountOnline",
      setAccountOffline: "kmd/setAccountOffline",
      openSuccess: "toast/openSuccess",
    }),
    async assignToCurrentNetwork() {
      if (this.account) {
        // add to current network automatically
        const info = { ...this.account };
        info.network = this.$store.state.config.env;
        await this.updateAccount({ info });
      }
    },
    async makeAssets() {
      this.assets = [];
      if (this.account && this.account.amount > 0) {
        this.assets.push({
          "asset-id": "",
          amount: this.account.amount,
          name: "ALG",
          decimals: 6,
          "unit-name": "",
        });
      }
      if (this.account && this.account.assets) {
        for (const accountAsset of this.account.assets) {
          if (!accountAsset["asset-id"]) continue;
          const asset = await this.getAsset({
            assetIndex: accountAsset["asset-id"],
          });
          if (asset) {
            this.assets.push({
              "asset-id": accountAsset["asset-id"],
              amount: accountAsset["amount"],
              name: asset["name"],
              decimals: asset["decimals"],
              "unit-name": asset["unit-name"],
            });
          }
        }
      }
    },
    getAssetSync(id) {
      const ret = this.$store.state.indexer.assets.find(
        (a) => a["asset-id"] == id
      );
      return ret;
    },
    getAssetName(id) {
      const asset = this.getAssetSync(id);
      if (asset) return asset["name"];
    },
    getAssetDecimals(id) {
      const asset = this.getAssetSync(id);
      if (asset) return asset["decimals"];
    },
    async reloadAccount() {
      await this.accountInformation({
        addr: this.$route.params.account,
      }).then(async (info) => {
        if (info) {
          this.updateAccount({ info });
          if (
            this.account &&
            this.account.rekeyedTo != this.account["auth-addr"]
          ) {
            const rekeyedTo = this.account["auth-addr"];
            console.error(
              `New rekey information detected: ${this.account.rekeyedTo} -> ${rekeyedTo}`
            );
            const info2 = {};
            info2.address = this.account.addr;
            info2.rekeyedTo = rekeyedTo;
            console.log("rekeyedTo", rekeyedTo);
            await this.updateAccount({ info: info2 });
            await this.openSuccess(
              `Information about rekeying to address ${rekeyedTo} has been stored`
            );
          }
        }
      });
      const searchData = await this.searchForTransactions({
        addr: this.$route.params.account,
      });
      if (searchData) {
        this.transactions = searchData.transactions;
      }
      console.log("this.transactions", this.transactions);
    },
    copyToClipboard(text) {
      if (copy(text)) {
        this.openSuccess(this.$t("global.copied_to_clipboard"));
      }
    },
    async deleteAccountClick() {
      await this.deleteAccount({
        name: this.account.name,
        addr: this.account.addr,
      });
      this.$router.push("/accounts");
    },
    async hideAccountClick() {
      if (this.account) {
        // add to current network automatically
        const info = { ...this.account };
        info.isHidden = !this.account.isHidden;
        await this.updateAccount({ info });
      }
    },
    sleep(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms));
    },
    async setAccountOnlineAtParticipationNode() {
      this.displayOnlineOfflineDialog = false;
      this.changeOnline = true;
      if (
        await this.setAccountOnline({
          account: this.$route.params.account,
          rounds: this.onlineRounds,
        })
      ) {
        await this.sleep(5000);
        this.changeOnline = false;
        await this.reloadAccount();
        this.openSuccess("You have set the account to online mode");
      } else {
        this.changeOnline = false;
      }
    },
    async setAccountOfflineAtParticipationNode() {
      this.displayOnlineOfflineDialog = false;
      this.changeOffline = true;
      if (
        await this.setAccountOffline({ account: this.$route.params.account })
      ) {
        await this.sleep(5000);
        this.changeOffline = false;
        await this.reloadAccount();
        this.openSuccess("You have set the account to offline mode");
      } else {
        this.changeOffline = false;
      }
    },
  },
};
</script>
