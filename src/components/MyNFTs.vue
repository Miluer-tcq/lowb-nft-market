<template>
  <div class="container">
    <div class="shadow px-5">
      <div class="row">
        <div class="col-6">
          <div class="css-cogtoi">{{ $t("lang.yourBSCWalletAddress") }}</div>
          <div class="css-94onap">{{$store.state.account}}</div>
        </div>
        <div class="col">
          <div class="css-cogtoi">{{ $t("lang.lowbBalance") }}:</div>
          <div class="css-94onap">{{$store.getters.lowb_balance}} LOWB</div>
        </div>
        <div class="col">
          <div class="css-cogtoi">{{ $t("lang.lowbMarketBalance") }}:</div>
          <div class="css-94onap">{{$store.getters.lowb_market_balance}} LOWB</div>
        </div>
      </div>
      <br>
      <div class="row">
        <h4>{{ $t("lang.depositAndWithdraw") }}</h4>
        <div class="input-group mb-3 col">
          <input type="number" class="form-control" @keyup="correct_toDeposit" v-model="toDeposit">
          <span class="input-group-text" >lowb</span>
          <button class="btn btn-primary" type="button" v-on:click="approve">{{ $t("lang.approve") }}</button>
          <button class="btn btn-primary active" type="button" v-on:click="deposit" :disabled="toDeposit<=0||$store.state.approvedBalance<toDeposit*1e18">{{ $t("lang.deposit") }}</button>
        </div>
        <div class="input-group mb-3 col">
          <input type="number" class="form-control" @keyup="correct_toWithdraw" v-model="toWithdraw">
          <span class="input-group-text">lowb</span>
          <button class="btn btn-primary active" type="button" id="button-addon2" v-on:click="withdraw">{{ $t("lang.withdraw") }}</button>
        </div>
      </div>
      <br>
      <div v-if="$store.state.chainId == '0x61'">
        <p>{{ $t("lang.goTo") }}<a href="https://testnet.binance.org/faucet-smart">Testnet Funds</a> {{ $t("lang.toGetMoreBNB") }}</p>
        <p>{{ $t("lang.youCan") }} <button @click = "claim">{{ $t("lang.claim") }}</button> {{ $t("lang.TenKLOWBThenRefreshtheWebpageAfterConfirmed") }}</p>
      </div>
    </div>
    <br>
    <h2>{{ $t("lang.NFTsOwned") }}</h2>
    <div class="row">
      <div v-for="nft in $store.state.myNfts" :key="nft.tokenId" class="col-sm-3">
        <b-card
          :title="$store.state.nftInfos[nft.groupId-1].name"
          :img-src="$store.state.nftInfos[nft.groupId-1].image"
          img-alt="Image"
          img-top
          tag="article"
          style="max-width: 20rem;"
          class="mb-2"
        >
          <h6 class="card-subtitle mb-2 text-muted">
            <div v-if="$store.state.nftInfos[nft.groupId-1].currentSupply<$store.state.nftInfos[nft.groupId-1].circulation">#{{nft.tokenId}}  [{{ $t("lang.new") }}]</div>
            <div v-else>#{{nft.tokenId}}  {{ $t("lang.circulation") }}: {{$store.state.nftInfos[nft.groupId-1].circulation}}</div>
          </h6>
          <div class="input-group mb-3">
            <input type="text" class="form-control" placeholder="0" v-model="toOffer[nft.tokenId]" @keyup="correct_toOffer(nft.tokenId)">
            <span class="input-group-text">lowb</span>
          </div>
          <div class="d-grid gap-2 d-md-flex justify-content-md-end">
            <button 
              class="btn btn-primary" 
              type="button" 
              v-on:click="approveNft(nft.tokenId, nft.groupId)" 
              :disabled="nft.isApproved || $store.state.nftInfos[nft.groupId-1].currentSupply<$store.state.nftInfos[nft.groupId-1].circulation">
              {{ $t("lang.approve") }}
            </button>
            <button 
              class="btn btn-primary active" 
              type="button" 
              v-on:click="offer(nft.tokenId, nft.groupId)" 
              :disabled="toOffer[nft.tokenId] <= 0 || !nft.isApproved || $store.state.nftInfos[nft.groupId-1].currentSupply<$store.state.nftInfos[nft.groupId-1].circulation">
              {{ $t("lang.offer") }}
            </button>
          </div>
          <div class="css-lvpxlc">
            <b-button block variant="outline-success" v-on:click="wrap(nft.tokenId)" >{{ $t("lang.wrap") }}</b-button>
          </div>
          <div class="css-lvpxlc">
            <div class="css-2x3sd8" v-if="$store.state.nftInfos[nft.groupId-1].currentSupply<$store.state.nftInfos[nft.groupId-1].circulation">
              <router-link :to="{path: '/new-token-details/'+nft.groupId}">{{ $t("lang.details") }}</router-link>
            </div>
            <div class="css-2x3sd8" v-else>
              <router-link :to="{path: '/token-details/'+nft.groupId}">{{ $t("lang.details") }}</router-link>
            </div>
          </div>
          
        </b-card>
      </div>
    </div>
    <div class="row" v-if="$store.state.myNfts.length==0">
      <br><p>{{ $t("lang.noNFTs") }}</p>
    </div>
    <br><br><br>
    <h2>{{ $t("lang.myBids") }}</h2>
    <div class="row">
      <div v-for="nft in $store.getters.loser_punks('my_bids')" :key="nft.id"  class="col-sm-3">
        <b-card
          :title="nft.name"
          :img-src="nft.image"
          img-alt="Image"
          img-top
          tag="article"
          style="max-width: 20rem;"
          class="mb-2"
        >
          <div class="m-t-10">
            <router-link :to="{path: '/token-details/'+nft.id}">{{ $t("lang.go") }} #{{nft.id}} {{ $t("lang.details") }}</router-link>
          </div>
        </b-card>
      </div>
    </div>
    <br>
    <div class="row" v-if="$store.getters.loser_punks('my_bids').length==0">
      <p>{{ $t("lang.noBids") }}</p>
    </div>
  </div>
</template>

<script>
import airdropFile from '../assets/AirdropClaim.json'
import { ethers } from "ethers";

export default {
  data: function() {
    return {
      toDeposit: 0,
      toWithdraw: 0,
      toOffer: []
    };
  },
  created () {
    console.log(this.$store.state.approvedBalance, this.toDeposit)
    this.$store.dispatch('filterPunks', 'my_bids')
  },
  methods: {
    correct_toDeposit: function () {
      if (this.toDeposit > Number(this.$store.getters.lowb_balance)) {
        this.toDeposit = this.$store.getters.lowb_balance
      }
      else if (this.toDeposit > 0) {
        this.toDeposit = Math.floor(this.toDeposit)
      }
      else {
        this.toDeposit = 0
      }
    },
    correct_toWithdraw: function () {
      if (this.toWithdraw > Number(this.$store.getters.lowb_market_balance)) {
        this.toWithdraw = this.$store.getters.lowb_market_balance
      }
      else if (this.toWithdraw > 0) {
        this.toWithdraw = Math.floor(this.toWithdraw)
      }
      else {
        this.toWithdraw = 0
      }
    },
    correct_toOffer: function (tokenId) {
      if (this.toOffer[tokenId] > 0) {
        this.toOffer[tokenId] = Math.floor(this.toOffer[tokenId])
      }
      else {
        this.toOffer[tokenId] = 0
      }
    },
    approve: function () {
      console.log("start approve")
      this.$store.dispatch('approveLowb', this.toDeposit)
    },
    deposit: function () {
      console.log("start deposit")
      this.$store.dispatch('depositLowb', this.toDeposit)
      this.toDeposit = 0
    },
    withdraw: function () {
      console.log("start withdraw")
      this.$store.dispatch('withdrawLowb', this.toWithdraw)
      this.toWithdraw = 0
    },
    // should remove when migrate to mainnet
    claim: async function () {
      console.log("claim 10k lowb")
      const airdropAbi = airdropFile['abi']
      const airdropAddress = '0xD80AF329e45BB88a333F0E5E2B6dF5Bde26b2736'
      const airdropContract = new ethers.Contract(airdropAddress, airdropAbi, global.provider);
      const airdropWithSigner = airdropContract.connect(global.signer);
      await airdropWithSigner.claim();
      console.log('claim 10000 lowb!');
    },
    approveNft: function (tokenId, groupId) {
      console.log("start approve nft")
      this.$store.dispatch('approveItemBid', {item: tokenId, group: groupId})
    },
    offer: function (tokenId, groupId) {
      console.log("start offer nft")
      this.$store.dispatch('offerItem', {id: tokenId, groupId: groupId, amount: this.toOffer[tokenId]})
      this.$set(this.toOffer, tokenId, 0)
    },
    wrap: function (tokenId) {
      console.log("start wrap nft")
      this.$store.dispatch('wrapItem', tokenId)
    },
  }
}
</script>

<style scoped>
.css-cogtoi {
    color: rgb(104, 104, 104);
    margin: 0px 0px 5px;
    font-size: 14px;
    line-height: 22px;
}
.css-94onap {
    margin: 0px;
    font-weight: 500;
    word-break: break-word;
    font-size: 18px;
    line-height: 26px;
}
.css-lvpxlc {
    display: flex;
    -webkit-box-align: center;
    align-items: center;
    padding: 5px 10px 10px;
}
.css-2x3sd8 {
    flex: 1 1 0%;
    display: flex;
    -webkit-box-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    align-items: center;
    height: 30px;
    font-size: 14px;
    font-weight: 500;
    background-color: rgba(227, 230, 238, 0.5);
    border-radius: 4px;
    cursor: pointer;
}
a {
    color: #ff04b4;
    text-decoration: none;
    font-weight: 700;
}
</style>