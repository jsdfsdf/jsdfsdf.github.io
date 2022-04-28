<script setup>
import { reactive, ref, computed } from 'vue'
import { ethers } from "ethers"
// import { create } from 'ipfs-http-client'
var provider, signer, contract, VoodooContract


// const client = create()
const states = reactive({
  hasMeta:false,
  isLogged:false,
  account:"",
  _to:"",
  msg:"",
  network:"",
  balance:0,
  checkBalance:0,
  balanceOf:0,
  look_address:""
})
const a = ref(-1)
const top_ten_recent = ref([
  // {address:"", num:0}
])
const hatest_look = ref([
  // {address:"", num:0}
])
const hatest = ref([
  // {address:"", num:0}
])
const new_messages = ref([
    // {address_from:"", address_to:"", msg:0}

])
const sent = ref([
    // {address:"", msg:0}

])
const received = ref([
    // {address:"", msg:0}

])
const VoodooAddress = "0xc5C4280884F91b3C998927D1DB69d951b9eB3723";

// The ERC-20 Contract ABI, which is a common contract interface
// for tokens (this is the Human-Readable ABI format)
const Voodoo = [
  // Some details about the token
    // "function name() view returns (string)",
    // "function symbol() view returns (string)",

  // Get the account balance
  // "function balanceOf(address) view returns (uint)",
  "function balanceOf(address account) external view returns (uint256)",

  // mint voodoo
  "function mint(address to, string memory newMessage) external",

  // Send some of your tokens to someone else
    // "function transfer(address to, uint amount)",

  // An event triggered whenever anyone transfers to someone else
    // "event Transfer(address indexed from, address indexed to, uint amount)"
    "event NewMessages(address indexed from, address indexed to, string newStr)"
  ];




if (typeof window.ethereum !== 'undefined') {
  console.log('MetaMask is installed!');
  states.hasMeta = true

}
else{
  states.hasMeta = false
}


async function connectMetamask() {

  provider = new ethers.providers.Web3Provider(window.ethereum)
  const x = await provider.send("eth_requestAccounts", [])
  states.isLogged = true
  signer = provider.getSigner()
  states.account = await signer.getAddress()
  await getNetwork()
  await getBalance()
  // The Contract object
  VoodooContract = new ethers.Contract(VoodooAddress, Voodoo, provider);
}

async function balanceOf() {
  states.balanceOf = await VoodooContract.balanceOf(states.checkBalance) //0xc5C4280884F91b3C998927D1DB69d951b9eB3723, 0x5b1a0eAa3bAC1327648Cbad633AfadF118Db7EA1
}

async function mint() {
  // await VoodooContract.mint(states._to, states.msg) 


  // The DAI Contract is currently connected to the Provider,
  // which is read-only. You need to connect to a Signer, so
  // that you can pay to send state-changing transactions.
  VoodooContract = VoodooContract.connect(signer);
  // a.value = await daiWithSigner.balanceOf(states._to) //0xc5C4280884F91b3C998927D1DB69d951b9eB3723, 0x5b1a0eAa3bAC1327648Cbad633AfadF118Db7EA1

  // a.value = await VoodooContract.estimateGas.mint(states._to, states.msg) 
  // Send 1 DAI to "ricmoo.firefly.eth"
  const tx = await VoodooContract.mint(states._to, states.msg);
}

async function getNetwork(){
  states.network = (await provider.getNetwork()).chainId //.getNetwork().chainId
  a.value = states.network.chainId
}

async function getBalance(){
  states.balance = await signer.getBalance()
}

// we need querry last month's event and get a dict which key is the address, value is count
async function getMyMessage(){
  top_ten_recent.value = []
  hatest_look.value = []
  const filterFrom = VoodooContract.filters.NewMessages();
  // const back = -3600 * 24 * 20
  const back = -3600  * 24 * 1/24
  const result = await VoodooContract.queryFilter(filterFrom, back)//, -3600 * 24 * 20)  // 1s  20 day 3600 * 24 * 20
  a.value =result
  const counts = {}
  const map_result = result.map(x => x.args[1])
  for (const event of map_result){
    counts[event] = counts[event] ? counts[event] + 1 : 1;
    
  }
  // a.value = result[0].args[1]
  a.value = counts
  for (const [key, value] of Object.entries(counts)) {
    top_ten_recent.value.push({address:key, num:value})
    hatest_look.value.push({address:key, num:value})
  }
  await most_hated()
}

// we need to use getbalance to get adress balance based on certain amount of list
async function most_hated(){
  hatest.value = []
  for (const address_count of hatest_look.value){
    hatest.value.push({address:address_count.address, num: await VoodooContract.balanceOf(address_count.address)}) 
  }
}


async function get_sent(){
  sent.value = []
  const filterFrom = VoodooContract.filters.NewMessages(states.look_address, null);
  // const back = -3600 * 24 * 20
  const back = -3600 * 24 * 1/24
  const result = await VoodooContract.queryFilter(filterFrom,back)//, -3600 * 24 * 20)  // 1s  20 day 3600 * 24 * 20
  // a.value =result

  for (const info of result) {
    sent.value.push({address:info.args[1], msg:info.args[2]})
  }
}


async function get_received(){
  received.value = []
  const filterFrom = VoodooContract.filters.NewMessages(null, states.look_address);
  // const back = -3600 * 24 * 20
  const back = -3600 * 24 * 1/24
  const result = await VoodooContract.queryFilter(filterFrom,back)//, -3600 * 24 * 20)  // 1s  20 day 3600 * 24 * 20
  // a.value =result

  for (const info of result) {
    received.value.push({address:info.args[0], msg:info.args[2]})
  }
}

async function see_chat(){
  await VoodooContract.on("NewMessages", (from, to, newStr) => {
 
    new_messages.value.push({address_from:from,address_to:to, msg:newStr})
    
});
}

const sortedTopTen = computed(() => {
  // return filtered todos based on
  // `todos.value` & `hideCompleted.value`
  const key="num"
  // return top_ten_recent.value
  const x = top_ten_recent.value
  return x.sort(function(a,b){
    return (b[key] -a[key] )

    })
})


const sortedHated = computed(() => {
  // return filtered todos based on
  // `todos.value` & `hideCompleted.value`
  const key="num"
  // return top_ten_recent.value
  return hatest.value.sort(function(a,b){
    return (b[key] -a[key] )
    })
  // return sortByKey(top_ten_recent, 'num')
})


// const filteredTodos = computed(() => {
//   return a.value > 0 ? 1:0
// })


// function sortByKey(array,key){
//   return array.value.sort(function(a,b){
//     var x=a[key];
//     var y=b[key];
//     return ((x<y)?-1:((x<y)?1:0));
//     });
//   }
</script>


<template>
  <!-- make this button work -->
  <!-- <button>count is: {{ count }}</button> -->
  <!-- <button @click="getBlockNum">getBlockNum {{blockN}}</button> -->
  <!-- <h1>has meta : {{ hasMeta }}</h1> -->
  <div class=wallet >  
    <div v-if="!states.isLogged">
      <button v-if="states.hasMeta" @click="connectMetamask">connect wallet now </button>
      <h1 v-else>Oh no Metamask 😢</h1>
    </div>
    
    <div v-else class="wallet">
      <p class="wallet">my address</p>
      <p class="wallet">{{states.account}}</p>
      <p class="wallet">chain Id: {{states.network}}</p>
      <a class="wallet" v-if="states.network != 80001"
       href="https://docs.polygon.technology/docs/develop/metamask/config-polygon-on-metamask/"> 
       follow guide to switch to Polygon's Mumbai-Testnet and go back refresh</a> 

    </div>
  </div>
  <!-- mint -->
  <div class=wallet v-if="states.isLogged">
    <input v-model="states._to" placeholder="Address here">
    <input v-model="states.msg" placeholder="msg here">
    <!-- <button @click="getNetwork" > {{a}}</button> -->
    <a class="wallet" v-if="states.balance == 0"
    href="https://faucet.polygon.technology/"> 
    use Polygon Faucet to get free test matic</a> 
    <button @click="mint" > mint </button>
    <input v-model="states.checkBalance" placeholder="Address here">
    <button @click="balanceOf" > balance: {{states.balanceOf}}</button>
    <button @click="getMyMessage">query leader board: {{a}}</button>
   </div>
  <div class=wallet>
    <p>top ten recent </p>
    <ul>
    <li v-for="todo in sortedTopTen" >
      {{ todo.address}} {{ todo.num}} 
    </li>
    </ul>
  </div>
  <div class=wallet>
    <p>top active ten history </p>
    <ul>
    <li v-for="todo in sortedHated">
      {{ todo.address}} {{ todo.num}} 
    </li>
    </ul>  
  </div>
  <div class=wallet>
    <h> look for people {{states.look_address}}</h>
    <input v-model="states.look_address" placeholder="Address here">
    <button @click="get_sent" > sent </button> 
    <ul>
    <li v-for="todo in sent" :key="todo.address">
      {{ todo.address}} {{ todo.msg}} 
    </li>
    </ul>  
    <button @click="get_received" > received </button>
    <ul>
    <li v-for="todo in received" :key="todo.address">
      {{ todo.address}} {{ todo.msg}} 
    </li>
    </ul>     

  </div>
  <img src="https://ipfs.io/ipfs/bafybeibfqywnrbtd5seoo33tzqyg5trqyjawt2d6uvr4hpjgid5ddaypqm/124.jpg"  width="500" height="500" style="border:5px solid black">
  <div>
    <button @click="see_chat">see_chat</button>
    <ul>
    <li v-for="todo in new_messages" :key="todo.num">
      {{ todo.address_from}} {{ todo.address_to}} {{ todo.msg}} 
    </li>
    </ul>  
  </div>
 


 





</template>
<style>
.wallet {
  margin-left: auto;
  margin-right: auto;
  width: 16em
}
</style>
