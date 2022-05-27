<template>
  <div class="crypto-container">
    <h3>{{ id + 1 }}</h3>
    <h3>{{ crypto.name }}</h3>
    <img :src="crypto.imageLink" alt="" />

<!-- if the crypto has pumped then give it a green color(class pump) -->
    <h3 :class="{ pump: pump, dump: !pump }">{{ crypto.change }} %</h3>
    <!-- when the market cap filter is chosen we show the market cap instead of the price -->
    <h3 class="price" v-if="marketCapFilter">
      {{ addSpaces(crypto.market_cap) }} {{ currencySymbol[filters.currency] }}
    </h3>
    <h3 class="marketCap" v-else>
      {{ addSpaces(crypto.price)
      }}{{ this.currencySymbol[this.filters.currency] }}
    </h3>
  </div>
</template>

<script>
export default {
  name: "CryptoCurrency",
  props: {
    crypto: Object,
    id: Number,
    filters: Object,
  },
  data() {
    return {
      pump: this.crypto.change > 0 ? true : false,
      marketCapFilter: false,
      currencySymbol: {
        usd: " $",
        eur: " €",
        czk: " kč",
      },
    };
  },
  methods: {
    addSpaces(input) {
      // add spaces so that a big value like 123456789 looks 123 456 789
      let newString = String(input);
      // do it only if the num is greater than 0
      if (input > 0) {
        let newStringArray = newString.split("");
        let leftovers = ''

        //if the number has decimal points then save it ot leftovers
        if(newStringArray.includes(".")){
          let dotIndex = newStringArray.indexOf('.')

          leftovers = newString.slice(dotIndex, newString.length)

          newStringArray = newStringArray.slice(0, dotIndex)

        }

        //now i add the spaces
        for (let i = newStringArray.length - 3; i > 0; i -= 3) {

          newStringArray.splice(i, 0, " ");
        }
        newString = newStringArray.join("");
        return newString + leftovers;
      }
      return newString;
    },
  },
  mounted() {
    if (this.filters.category === "Market Cap") {
      this.marketCapFilter = true;
    } else {
      this.marketCapFilter = false;
    }
  },
};

//if + green
</script>


<style scoped>
h3 {
  white-space: nowrap;
}
.crypto-container {
  display: grid;
  align-items: center;
  grid-template-columns: repeat(5, 1fr);

  background-color: #8e9dcc;
  border: 1px solid #f9f9ed;
  border-radius: 10px;

  max-width: 700px;
  padding: 20px;
  margin: 20px auto;

  color: #f9f9ed;
}

.crypto-brand,
.statistics {
  display: flex;
  align-items: center;
  gap: 20px;
}

.crypto-brand > img {
  width: 50px;
  height: 50px;
}

.pump {
  color: #00ff2a;
}

.pump::before {
  content: "+";
  color: #00ff2a;
}

.dump {
  color: #f22b2b;
}
</style>