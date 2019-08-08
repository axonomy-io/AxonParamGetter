<template>
  <div class="home">
    <h2>AXON合约参数查询</h2>
    <dl class="show-detailed token-detailed">
      <dt>Token详情</dt>
      <dd v-for="(e, i) in tokenDetailed">
        <span :class="[e.icon, 'left']">{{ e.key }}</span>
        <span class="right">{{ e.val }}</span>
      </dd>
    </dl>
    <dl class="show-detailed user-detailed">
      <dt>用户详情</dt>
      <dd v-for="(e, i) in accountDetailed">
        <span :class="[e.icon, 'left']">{{ e.key }}</span>
        <span class="right">{{ e.val }}</span>
      </dd>
    </dl>
    <dl class="show-detailed user-detailed">
      <dt>本Dapp已开源， 源码地址:</dt>
      <dd>
        <span class="left">github</span>
        <a href="https://github.com/axonomy-io/AxonParamGetter" class="right">https://github.com/axonomy-io/AxonParamGetter</a>
      </dd>
    </dl>
    <div class="tips-box" v-show="isLoadding">
      <div class="loading">
        <span></span>
        <span></span>
        <span></span>
        <span></span>
        <span></span>
      </div>
      <p class="tips-loading">正在查询合约，请稍候  ☺...</p>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import AxonToken from '@/assets/abi/AxonToken.json';
import '@/assets/js/truffle-contract.js';
const web3 = (window as any).web3;
if (!web3) {
  const tips = document.createElement('div');
  tips.className = 'tips-box';
  tips.innerHTML = '<p class="tips">需要MetaMask，点击<a href="https://metamask.io/" target="_blank">https://metamask.io/</a>安装</p>';
  document.body.appendChild(tips);
}
const TruffleContract = (window as any).TruffleContract;
const contractsAxonToken = TruffleContract(AxonToken);
contractsAxonToken.setProvider(web3.currentProvider);
@Component
export default class Home extends Vue {
  private netIdName: string = 'Foundation';
  private addressesAxonToken: string = '';
  private accountDetailed: object[] = [];
  private tokenDetailed: object[] = [];
  private isLoadding: boolean = false;

  public async created() {
    await web3.currentProvider.enable();
    this.isLoadding = true;
    const network = web3.version.network;
    switch (network) {
      case '1': // 主网络
        this.netIdName = 'Foundation';
        this.addressesAxonToken = '0xe38168d5d51b2373fa0ef76bcbe85f8bcf63a1bc';
        break;
      case '3': // Ropsten
        this.netIdName = 'Ropsten';
        this.addressesAxonToken = '0xd0027387d7119671cfa761085E6C5628BE33EC38';
        break;
      case '42': // Kovan
        this.netIdName = 'Kovan';
        this.addressesAxonToken =  '0xeB740Ee7D1F58eFb7dCa1DC1d169672d58A46CD9';
        break;
      case '4': // Rinkeby
        this.netIdName = 'Rinkeby';
        this.addressesAxonToken =  '';
        break;
      default:
        this.netIdName = 'Unknown';
    }
    const tempToken: any[] = [];
    const tempAccount: any[] = [];
    const axonToken = await contractsAxonToken.at(this.addressesAxonToken);
    const cap = await axonToken.cap();
    const capTotal = cap.toNumber() / Math.pow(10, 18);
    const name = await axonToken.name();
    const symbol = await axonToken.symbol();
    const decimals = (await axonToken.decimals()).toNumber();
    const total = (await axonToken.totalSupply()) / Math.pow(10, 18);
    const currentDifficulty = (await axonToken.get_current_difficulty()).toNumber() / Math.pow(10, 18);
    const investMined = (await axonToken.get_invest_mined()).toNumber();
    const account = await this.getAccounts();
    const balanceOf = await axonToken.balanceOf(account);
    const balanceOfDetailed = (balanceOf.toNumber() / Math.pow(10, 18)).toFixed(3);
    tempToken.push(
      {key: '合约地址', val: this.addressesAxonToken, icon: 'international'},
      {key: '代币名称', val: name, icon: 'chart'},
      {key: '代币符号', val: symbol, icon: 'icon'},
      {key: '代币总量', val: capTotal, icon: 'documentation'},
      {key: '代币已发行量', val: total, icon: 'documentation'},
      {key: '代币精度', val: decimals, icon: 'nested'},
      {key: '挖矿难度', val: currentDifficulty, icon: 'nested'},
      {key: '挖矿进度', val: investMined / Math.pow(10, 18), icon: 'guide'},
    );
    tempAccount.push(
      {key: '当前网络', val: this.netIdName, icon: 'dashboard'},
      {key: '当前Public Key', val: account, icon: 'peoples'},
      {key: '代币持有量', val: balanceOfDetailed, icon: 'peoples'},
    );
    this.accountDetailed = tempAccount;
    this.tokenDetailed = tempToken;
    this.isLoadding = false;
  }
  private async getAccounts() {
    return new Promise((resolve, reject) => {
      web3.eth.getAccounts((error: object, accounts: object[]) => {
        if (error) {
          reject(error);
        }
        resolve(accounts[0]);
      });
    });
  }
}
</script>
<style lang="less">
*{
  margin: 0;
  padding: 0;
}
h2{
  line-height: 40px;
  padding: 20px 0;
}
.show-detailed{
  width: 80%;
  margin: 40px 10%;
  dt{
    text-align: left;
    box-sizing: border-box;
    white-space: normal;
    padding-left: 10px;
    font-weight: bold;
    color: #909399;
    line-height: 24px;
    font-size: 14px;
    border-collapse: separate;
    padding: 10px 10px;
  }
  dd{
    list-style: none;
    display: flex;
    border: 1px solid #dcdfe6;
    border-bottom: 0;
    line-height: 36px;
    span{
      text-align: left;
      padding-left: 20px;
    }
    .left{
      flex: 1;
      background: #f5f7fa;
      border-right: 1px solid #dcdfe6;
    }
    .right{
      flex: 4;
    }
    a{
      color: #409EFF;
      text-align: left;
      padding-left: 20px;
    }
  }
  dd:last-child{
    border-bottom: 1px solid #dcdfe6;
  }
}
.user-detailed{
  
}
.tips-box{
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  position: fixed;
  background: rgba(0, 0, 0, 0.6);
  .tips{
    padding-top: 200px;
    text-align: center;
    color: #FFF;
    a{
      color: #FFF
    }
  }
  .loading{
    position: fixed;
    width: 80px;
    height: 40px;
    left: 50%;
    top: 50%;
    margin: -100px 0 0 -40px;
  }
  .tips-loading{
    line-height: 40px;
    text-align: center;
    position: fixed;
    width: 200px;
    left: 50%;
    top: 50%;
    font-weight: bold;
    color: #FFF;
    margin: -20px 0 0 -100px;
  }
  .loading span{
    display: inline-block;
    width: 8px;
    height: 100%;
    margin: 0 3px;
    border-radius: 4px;
    background: lightgreen;
    -webkit-animation: load 1s ease infinite;
  }
  @-webkit-keyframes load{
    0%,100%{
        height: 40px;
        background: lightgreen;
    }
    50%{
        height: 70px;
        margin: -15px 3px;
        background: lightblue;
    }
  }
  .loading span:nth-child(2){
    -webkit-animation-delay:0.2s;
  }
  .loading span:nth-child(3){
    -webkit-animation-delay:0.4s;
  }
  .loading span:nth-child(4){
    -webkit-animation-delay:0.6s;
  }
  .loading span:nth-child(5){
    -webkit-animation-delay:0.8s;
  }
}
</style>
