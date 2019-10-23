<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <button v-if="!account['me']" v-on:click="createRadixIdentity('me')">create my identity</button>
    <button
      v-if="!account['friend']"
      v-on:click="createRadixIdentity('friend')"
    >create friend Identity</button>

    <!-- <button v-on:click="sendAtom(1,'white')">Send atom</button> -->
    <!-- <button v-on:click="createToken">createToken</button> -->

    <button v-on:click="test">TEST</button>

    <div class="row">
      <div class="column" v-if="account['me']">
        <ul>
          <b>MY ADR:</b>
          <button v-if="account['me']" v-on:click="recreateIdentity('me')">recreate me</button>
          <li>{{account["me"].getAddress() }}</li>
        </ul>

        <ul>
          <b>BALANCES:</b>
          <button v-if="account['me']" v-on:click="getMoneyFromFaucet('me')">Moar XRD</button>
          <li v-for="(value, key) in balances['me']" v-bind:key="key">{{ key }}: {{ value }}</li>
        </ul>

        <ul>
          <b>TRANSACTIONS:</b>
          <li
            v-for="(transaction, aid) in transactions['me']"
            v-bind:key="aid"
          >{{transaction.timestamp}} # {{ transaction.value }} {{ transaction.token }}</li>
        </ul>

        <ul>
          <b>MESSAGES:</b>
          <li v-for="(message, aid) in messages['me']" v-bind:key="aid">{{message}}</li>
        </ul>

        <b>SEND TOKEN:</b>
        <br />
        <select v-model="makeTransfer.from">
          <option v-if="account['me']">me</option>
          <option v-if="account['friend']">friend</option>
        </select>
        <input v-model="makeTransfer.amount" placeholder="amount" />
        <select v-if="makeTransfer.from" v-model="makeTransfer.token">
          <option
            v-for="token in allTokens[makeTransfer.from]"
            v-bind:key="token"
            v-bind:value="token"
            selected
          >{{_getSymbolfromKey(token)}}</option>
        </select>
        <input v-model="makeTransfer.toAdr" placeholder="toAdr" />
        <button
          v-on:click="sendToken(makeTransfer.from,makeTransfer.amount,makeTransfer.token,makeTransfer.toAdr)"
        >send</button>

        <br />
        <br />
        <b>CREATE TOKEN:</b>
        <br />

        <select v-model="makeToken.from">
          <option v-if="account['me']">me</option>
          <option v-if="account['friend']">friend</option>
        </select>
        <input v-model="makeToken.symbol" placeholder="symbol" />
        <br />
        <input v-model="makeToken.name" placeholder="name" />
        <input v-model="makeToken.description" placeholder="description" />
        <br />
        <input v-model="makeToken.granularity" placeholder="granularity" />
        <input v-model="makeToken.amount" placeholder="amount" />
        <br />

        <button
          v-on:click="createToken(makeToken.from, makeToken.symbol,makeToken.name,makeToken.description,makeToken.granularity,makeToken.amount)"
        >create</button>
      </div>
      <div class="column" v-if="account['friend']">
        <ul>
          <b>FRIEND ADR:</b>
          <button v-if="account['friend']" v-on:click="recreateIdentity('friend')">recreate friend</button>
          <li>{{account['friend'].getAddress() }}</li>
        </ul>

        <ul>
          <b>BALANCES:</b>
          <button v-if="account['friend']" v-on:click="getMoneyFromFaucet('friend')">moar XRD</button>
          <li v-for="(value, key) in balances['friend']" v-bind:key="key">{{ key }}: {{ value }}</li>
        </ul>

        <ul>
          <b>TRANSACTIONS:</b>
          <li
            v-for="(transaction, aid) in transactions['friend']"
            v-bind:key="aid"
          >{{transaction.timestamp}} # {{ transaction.value }} {{ transaction.token }}</li>
        </ul>

        <ul>
          <b>MESSAGES:</b>
          <li v-for="(message, aid) in messages['friend']" v-bind:key="aid">{{message}}</li>
        </ul>
      </div>
    </div>

    <!-- end .canvas -->
  </div>
</template> 

<script>
import {
  radixUniverse,
  RadixUniverse,
  radixTokenManager,
  RadixKeyStore,
  RadixSimpleIdentity,
  RadixIdentityManager,
  RadixAccount,
  RadixTransactionBuilder,
  RRI
} from "radixdlt";

import moment from "moment";
import { Decimal } from "decimal.js";

export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  components: {},
  data() {
    return {
      identity: { me: null, friend: null },
      account: { me: null, friend: null },
      messages: { me: [], friend: [] },
      balances: { me: {}, friend: {} },
      transactions: { me: [], friend: [] },
      allTokens: { me: null, friend: null },
      makeTransfer: {
        amount: null,
        token: null,
        toAdr: null,
        from: null
      },
      makeToken: {
        from: null,
        symbol: null,
        name: null,
        description: null,
        granularity: null,
        amount: null
      }
    };
  },
  methods: {
    loadIdentity(key) {
      return new Promise((resolve, reject) => {
        // NOTE: This is insecure, normally you would ask the user for a password
        const password = "SuperDuperSecretPassword";
        const identityManager = new RadixIdentityManager();

        if (!localStorage["keystore_" + key]) {
          // Generate a new random identity
          const identity = identityManager.generateSimpleIdentity();
          RadixKeyStore.encryptKey(identity.address, password)
            .then(encryptedKey => {
              localStorage["keystore_" + key] = JSON.stringify(encryptedKey);
              console.log("Encrypted private key stored in localstorage");
            })
            .catch(error => {
              console.error("Error encrypting private key", error);
            });
          resolve(identity);
        } else {
          // Load identity from localstorage
          const encryptedKey = JSON.parse(localStorage["keystore_" + key]);
          RadixKeyStore.decryptKey(encryptedKey, password)
            .then(address => {
              console.log("Private key successfuly decrypted");
              const identity = new RadixSimpleIdentity(address);
              return resolve(identity);
            })
            .catch(error => {
              console.error("Error decrypting private key", error);
            });
        }
      });
    },
    createRadixIdentity(who) {

      console.log(who);

      this.loadIdentity(who).then(identity => {
        this.identity[who] = identity;
        this.account[who] = identity.account;
        this.account[who].openNodeConnection();
        console.log("My account address: ", identity.account.getAddress());

        // this.account[who].dataSystem
        //   .getApplicationData("my-test-app")
        //   .subscribe(data => {
        //     this.messages[who].push(JSON.parse(data.data.payload.data));
        //   });

        this.account[who].messagingSystem.getAllMessages().subscribe(
            data => {
                this.messages[who].push(data.message.content)
            }
        )

        // Subscribe for all previous transactions as well as new ones
        this.account[who].transferSystem
          .getAllTransactions()
          .subscribe(transactionUpdate => {
            console.log(transactionUpdate.transaction);

            const allKeys = Object.keys(
              transactionUpdate.transaction.tokenUnitsBalance
            );

            allKeys.map(key => {
              const token = key.split("/")[2];
              const curValue = transactionUpdate.transaction.tokenUnitsBalance[
                key.toString()
              ].toString();
              //   this.$set(this.transactions,token,curValue)s

              const timestampString = moment(
                transactionUpdate.transaction.timestamp
              ).format("DD.MM.YY H:mm:ss");
              this.transactions[who].push({
                aid: transactionUpdate.transaction.aid,
                timestamp: timestampString,
                token: token,
                value: curValue
              });
              //   console.log("balance is: " + curBalance + " " + token);
            });

            console.log(this.transactions[who]);
          });

        this.account[who].transferSystem
          .getTokenUnitsBalanceUpdates()
          .subscribe(balance => {
            const allKeys = Object.keys(balance);

            this.allTokens[who] = allKeys;

            allKeys.map(key => {
              const token = key.split("/")[2];
              const curBalance = balance[key.toString()].toString();
              this.$set(this.balances[who], token, curBalance);
            });
          });
      });
    },
    _getSymbolfromKey(key) {
      return key.split("/")[2].toString();
    },
    recreateIdentity(who) {
      localStorage.removeItem("keystore_" + who);
      this.createRadixIdentity(who);
    },

    sendAtom(coord = "aff", color = "black") {
      const applicationId = "aff";

      //   const payload = JSON.stringify(coord);

      const toAccount = RadixAccount.fromAddress(
        "JHWiQH3hDoKi31WxJZfp8S5js8P9hvo6BiHk3pGu4xQEHYtWN6J",
        true
      );

      for (let index = 0; index < 100; index++) {
        const transactionStatus = RadixTransactionBuilder.createPayloadAtom(
          this.account["me"],
          [toAccount],
          applicationId,
          "payload",
          false
        ).signAndSubmit(this.identity["me"]);

        transactionStatus.subscribe({
          next: status => {
            console.log(status);
            // For a valid transaction, this will print, 'FINDING_NOD   E', 'GENERATING_POW', 'SIGNING', 'STORE', 'STORED'
          },
          complete: () => {
            console.log("Transaction complete");
          },
          error: error => {
            console.error("Error submitting transaction", error);
          }
        });
      }
    },
    sendToken(who, amount_input, token, toAdr) {
      // No need to load data from the ledger for the recipient account
      const toAccount = RadixAccount.fromAddress(toAdr, true);

      const balance = new Decimal(
        this.balances[who][this._getSymbolfromKey(token)]
      );
      const amount = new Decimal(amount_input);

      if (!amount.greaterThan(balance)) {
        const transactionStatus = RadixTransactionBuilder.createTransferAtom(
          this.account[who],
          toAccount,
          token,
          amount
        ).signAndSubmit(this.identity[who]);

        transactionStatus.subscribe({
          next: status => {
            console.log(status);
            // For a valid transaction, this will print, 'FINDING_NODE', 'GENERATING_POW', 'SIGNING', 'STORE', 'STORED'
          },
          complete: () => {
            console.log("Transaction complete");
          },
          error: error => {
            console.error("Error submitting transaction", error);
          }
        });
      } else {
        console.log("insufficient funds");
      }
    },
    getMoneyFromFaucet(who) {
      // betanet emulator adr: JH1P8f3znbyrDj8F4RWpix7hRkgxqHjdW2fNnKpR3v6ufXnknor
      // hosted bn adr: 9ecjMNCFDSbLZxVpfbFwFTLWuL7SH3Q49uzGrpK3bUcze6CJtDr

      const faucetAddress =
        "9ecjMNCFDSbLZxVpfbFwFTLWuL7SH3Q49uzGrpK3bUcze6CJtDr";
      const faucetAccount = RadixAccount.fromAddress(faucetAddress, true);
      const message = "yo mf, wheres my money?";

      RadixTransactionBuilder.createRadixMessageAtom(
        this.account[who],
        faucetAccount,
        message
      ).signAndSubmit(this.identity[who]);

      const radixToken = radixUniverse.nativeToken;
      this.account[who].transferSystem
        .getTokenUnitsBalanceUpdates()
        .subscribe(balance => {
          // Get the balance for the token we are interested in
          const nativeTokenBalance = balance[radixToken.toString()];
          console.log("balance is: " + nativeTokenBalance + " XRD");
        });
    },
    createToken(who, symbol, name, description, granularity, amount) {
      const iconUrl = "http://a.b.com/icon.png";

      new RadixTransactionBuilder()
        .createTokenSingleIssuance(
          this.account[who],
          name,
          symbol,
          description,
          granularity,
          amount,
          iconUrl
        )
        .signAndSubmit(this.identity[who])
        .subscribe({
          next: status => {
            console.log(status);
          },
          complete: () => {
            console.log("Token defintion has been created");
          },
          error: error => {
            console.error("Error submitting transaction", error);
          }
        });
    },
    test() {
      const account = RadixAccount.fromAddress(
        "9ecjMNCFDSbLZxVpfbFwFTLWuL7SH3Q49uzGrpK3bUcze6CJtDr"
      );
      account.openNodeConnection();

      account.transferSystem.balance; // This is the account balance
      account.transferSystem.tokenUnitsBalance; // This is the account balance in token units
      account.transferSystem.transactions; // This is a list of transactions

      // Subscribe for any new incoming transactions
      account.transferSystem.transactionSubject.subscribe(transactionUpdate => {
        console.log(transactionUpdate);
      });

      // Subscribe for all previous transactions as well as new ones
      account.transferSystem
        .getAllTransactions()
        .subscribe(transactionUpdate => {
          console.log(transactionUpdate);
        });
    }
  },
  created() {
      // Bootstrap the universe
      radixUniverse.bootstrap(RadixUniverse.BETANET_EMULATOR);
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin: 0 10px;
}
a {
  color: #42b983;
}
.canvas {
  /* Perfectly square */
  width: 770px;
  height: 770px;
}

.pixelLight {
  /* We'll need 64 total pixels in our HTML */
  width: 15px;
  height: 15px;
  float: left;
  box-shadow: 0px 0px 1px #42b983;
  /* background-color: rgb(255, 230, 0); */
}

.pixelDark {
  /* We'll need 64 total pixels in our HTML */
  width: 15px;
  height: 15px;
  float: left;
  box-shadow: 0px 0px 1px #42b983;
  background-color: #000;
}

.small {
  font-size: 8px;
  color: #fff;
}
.column {
  float: left;
  width: 50%;
}

/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}
</style>
