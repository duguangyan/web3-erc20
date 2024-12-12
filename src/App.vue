
<script setup>
import { computed, onMounted, ref } from 'vue'
import Web3 from 'web3'

import mtcJSON from './contracts/MyToken.json'

const web3 = new Web3(Web3.givenProvider || 'https://linea-sepolia.infura.io/v3/638355f30f124b0daf4131933304abd0');

// console.log(web3)
const mtcContract = new web3.eth.Contract(mtcJSON.abi, '0x68BF7AaD15b1bbEC675c90DBB1b56524a8F3ccF0')
// console.log(mtcContract)

// 代币信息
const name = ref('')
const symbol = ref('')
const decimals = ref('') 
const totalSupply = ref('')
const balanceOf = ref('')
const currentAccount = ref('')

const getCoinInfo = async () => {
  const account = await web3.eth.requestAccounts();
  console.log(account)
  currentAccount.value = account
  name.value = await mtcContract.methods.name().call()
  symbol.value = await mtcContract.methods.symbol().call()
  decimals.value = await mtcContract.methods.decimals().call()
  totalSupply.value = await mtcContract.methods.totalSupply().call()

  balanceOf.value = await mtcContract.methods.balanceOf(account[0]).call()

}

const ts = computed(() => {
  return web3.utils.fromWei(totalSupply.value, 'ether')
})


const bo = computed(() => {
  return web3.utils.fromWei(String(balanceOf.value), 'ether')
})


const toAddress = ref('')
const amount = ref('')

const send = async () => {
  // const account = await web3.eth.requestAccounts();
  // console.log(account)
  // const tx = await mtcContract.methods.transfer(toAddress.value, web3.utils.toWei(amount.value, 'ether')).send({from: account[0]})
  // console.log(tx)


  const weiAmount = web3.utils.toWei(amount.value, 'ether')


  mtcContract.methods
  .transfer(toAddress.value, weiAmount)
  .send({from: currentAccount.value[0]})
  .on("receipt",(res)=>{
    console.log("交易成功",res)
  })

}



onMounted(() => {
  getCoinInfo()
})

</script>

<template>
  <h1>RABTC基本信息：</h1>
  <hr>
  <p>当前账户: {{ currentAccount }}</p>
  <p>代币名称：{{ name }}</p>
  <p>代币标识：{{ symbol }}</p>
  <p>小数位数：{{ decimals }}</p>
  <p>发行量：{{ ts }}</p>
  <p>持有数量：{{ bo }}</p>
  <h1>转账操作：</h1>
  <hr>
  <p>转入地址: <input style="width: 600px" type="text" v-model="toAddress" /></p>
  <p>转出金额: <input type="text" v-model="amount" /> RABTC</p>

  <button @click="send">开始转账</button>
</template>


<style lang="less">

</style>
