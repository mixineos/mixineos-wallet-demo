<template>
  <div class="hello">
    <h1>mixineos wallet demo</h1>

    <p>{{currentAccount}}</p>

    <button @click="connect">connect</button>

    <button @click="login">login</button>

    <button @click="logout">logout</button>

    <br />

    <button @click="account">account('eos')</button>
    <button @click="transfer()">transfer</button>
  </div>
</template>

<script>
import { InitWallet } from "mixineos-wallet"
const members = [
    "e07c06fa-084c-4ce1-b14a-66a9cb147b9e",
    "e0148fc6-0e10-470e-8127-166e0829c839",
    "18a62033-8845-455f-bcde-0e205ef4da44",
    "49b00892-6954-4826-aaec-371ca165558a"
];

InitWallet({
  node_url: "https://api.eosn.io",
  client_id: "d78a6e9e-5d23-4b24-8bf3-05dc8576cf8b", //"3e72ca0c-1bab-49ad-aa0a-4d8471d375e7",
  mainContract: "mixincrossss",
  mixinWrapTokenContract: "mixinwtokens",
  contractProcessId: "e0148fc6-0e10-470e-8127-166e0829c839",
  members,
  debug: false,
  inject: true
});

import ScatterJS from "@scatterjs/core";
import ScatterEOS from "@scatterjs/eosjs2";
import { JsonRpc, Api } from "eosjs";

// import { InitWallet } from "mixineos-multisig-wallet"
// import VConsole from "vconsole";
// var vConsole = new VConsole();

const network = ScatterJS.Network.fromJson({
  blockchain: "eos",
  chainId: "aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906",
  host: "api.eosn.io",
  port: 443,
  protocol: "https"
});

const rpc = new JsonRpc(network.fullhost());

const requiredFields = { accounts: [network] };

let scatter = "";
let api = "";

ScatterJS.plugins(new ScatterEOS()); 

export default { 
  name: "HelloWorld",
  data() {
    return {
      currentAccount: "",
      currentPermission: "",
      currentPublicKey: ""
    };
  },
  created() {
    ScatterJS.scatter.connect("scatter-demo", { network }).then(connected => {
      if (!connected) {
        alert("no connect");
        return false;
      }
      alert("connected");
      scatter = ScatterJS.scatter;
    });
  },

  methods: {
    connect: function() {
        // let args = {
        //     'from': 'baaaaaaaamvm',
        //     'to': 'learnforever',
        //     'quantity': "0.00000001 MEOS",
        //     'memo': 'hello,world'
        // }

        // window.mixineos.pushAction("mixinwtokens", "transfer", args);
      ScatterJS.scatter.connect("scatter-demo").then(connected => {
        if (!connected) {
          alert("no connect");
          return false;
        }

        alert("connected");
        scatter = ScatterJS.scatter;
      });
    },
    account: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      alert(JSON.stringify(scatter.account("eos")));
    },
    transfer: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      api
        .transact(
          {
            actions: [
              {
                account: "mixinwtokens",
                name: "transfer",
                authorization: [
                  {
                    actor: this.currentAccount,
                    permission: this.currentPermission
                  }
                ],
                data: {
                  from: this.currentAccount,
                  to: "mixincrossss",
                  quantity: "0.00010000 MEOS",
                  memo: "test mixin wallet"
                }
              }
            ]
          },
          {
            blocksBehind: 3,
            expireSeconds: 30
          }
        )
        .then(res => { 
          alert("yes" + res);
        })
        .catch(err => {
          alert("fail" + err);
          console.log("fail" + JSON.stringify(err));
        });
    },
    logout: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      scatter.logout().then(a => alert(a));
    },
    login: function() {
      scatter
        .login()
        .then(id => {
          const account = scatter.identity.accounts.find(
            x => x.blockchain === "eos"
          );

          this.currentAccount = account.name;
          this.currentPermission = account.authority;
          this.currentPublicKey = account.publicKey;

          alert("get account:" + JSON.stringify(account));

          api = scatter.eos(network, Api, { rpc });
        })
        .catch(error => {
          alert("get identity error");
          console.error(error);
        });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

button {
  margin: 10px;
  padding: 8px 20px;
  background-color: #efefef;
  border-radius: 30px;
  outline: none;
  border: none;
}
</style>
