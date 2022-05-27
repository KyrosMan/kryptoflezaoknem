<template>
<CryptoCurrency 
    v-for="(coin, index) in cryptocurrencies[0]"
    :key="coin.id"
    :crypto="cryptocurrencies[0][index]"
    :filters="filters"
    :id="index"
  />
</template>

<script>
import CryptoCurrency from "./CryptoCurrency"

export default {
    name: 'CryptoConainer',
    components:{
        CryptoCurrency
    },
    props: {
        filters: Object,
        cryptocurrencies: Object
    },

}

</script>
