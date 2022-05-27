<template>
  <filterContainer
    @filter-chosen="filterRefresh"
    :filters="filtersData"
    :key="mainKey"
  />

  <CryptoContainer
    :key="mainKey"
    :cryptocurrencies="cryptocurrencies"
    :filters="filters"
  />

  <LoaderElement :text="errorMessage" v-if="isLoading" />
</template>

<script>
import FilterContainer from "./components/filterContainer.vue";
import CryptoContainer from "./components/CryptoContainer.vue";
import LoaderElement from "./components/LoaderEl.vue";

export default {
  name: "App",
  components: {
    FilterContainer,
    CryptoContainer,
    LoaderElement,
  },
  data() {
    return {
      // currently used filters
      filters: {
        currency: "usd",
        category: "Price",
        interval: "24h",
      },
      // each filter has a name and a key
      filterKeys: {
        USD: "usd",
        EUR: "eur",
        CZK: "czk",
        Price: "Price",
        "Top Risers": "Risers",
        "Top Failers": "Failers",
        "Market Cap": "Market Cap",
        "24h": "1",
        "7d": "7",
        "30d": "30",
        "90d": "90",
        "180d": "180",
        "1y": "365",
        "5y": "1825",
      },
      mainKey: 0, // i use mainKey++ to rerender the containers when a button is clicked
      isLoading: false,
      errorMessage: "",
    };
  },
  mounted() {
    // render the default values
    this.filterRefresh(0, "USD");
  },
  methods: {
    // each time a filter button is clicked i refresh the 2 containers
    async filterRefresh(index, value) {
      this.isLoading = true;
      // the filter container is divided into 3 parts each with and index starting from 0
      // i add the value passed on from the button to the active filters array
      switch (index) {
        case 0:
          if (this.filters.currency != this.filterKeys[value]) {
            this.filters.currency = this.filterKeys[value];
          }
          break;
        case 1:
          if (this.filters.category != this.filterKeys[value]) {
            this.filters.category = this.filterKeys[value];
          }
          break;
        case 2:
          if (this.filters.interval != this.filterKeys[value]) {
            this.filters.interval = this.filterKeys[value];
          }
          break;
      }

      // active property is used to change styles of buttons  
        for (let i = 0; i < this.filtersData[index].length; i++) {
          this.filtersData[index][i].active = false;
          if (this.filtersData[index][i].key === value) {
            this.filtersData[index][i].active = true;
          }
        }
        
      

      //rerender after getPrice() is done
      await this.getPrice(index, value);
      this.mainKey++;
    },

    async getPrice(filterType, value) {
      // when errorMeassage is "" it means that there has been an error - too many requests and the website is waiting
      if (this.errorMessage === "") {
        try {
          switch (filterType) {
            case 0:
              for (let i = 0; i < this.cryptocurrencies[0].length; i++) {
                let cryptoKeyword = this.cryptocurrencies[0][i].keyword;
                const newPrice = await fetch(
                  `https://api.coingecko.com/api/v3/simple/price?ids=${cryptoKeyword}&vs_currencies=${this.filters.currency}`
                );
                const results = await newPrice.json();

                let currentCrypto = results[cryptoKeyword];

                let finalPrice = currentCrypto[this.filters.currency];

                this.cryptocurrencies[0][i].price = finalPrice;
              }
              // sort and update % change
              this.sortCryptos(this.filterKeys[value]);
              this.getPrice(2, this.filters.interval);
              break;

            case 1:
              this.sortCryptos(this.filterKeys[value]);
              break;

            case 2: {
              //get todays and a day in the past
              let today = new Date();
              today.setDate(today.getDate() - this.filterKeys[value]);
              let pastDay =
                today.getDate() +
                "-" +
                (today.getMonth() + 1) +
                "-" +
                today.getFullYear();

              for (let i = 0; i < this.cryptocurrencies[0].length; i++) {
                let cryptoKeyword = this.cryptocurrencies[0][i].keyword;

                // get a new Price from the past to compare 
                const newPrice = await fetch(
                  `https://api.coingecko.com/api/v3/coins/${cryptoKeyword}/history?date=${pastDay}&localization=false`
                );
                const results = await newPrice.json();

                this.cryptocurrencies[0][i].market_cap = Math.floor(
                  results.market_data.market_cap[this.filters.currency]
                );

                let oldPrice =
                  results.market_data.current_price[this.filters.currency];
                // compare the 2 prices to get the % of growth
                this.cryptocurrencies[0][i].change = Math.floor(
                  100 * (this.cryptocurrencies[0][i].price / oldPrice) - 100
                );
              }
              
              this.sortCryptos(this.filters.category);
              break;
            }
          }
          //now the loading screen will disable
          this.isLoading = false;
        } catch (e) {
          // if too many requests are sent the API stops responding wich creates an error
          // in the case of error the website throws the loading screen and puts it away 30 seconds later
          console.log(e);
          this.isLoading = true;
          this.errorMessage =
            "You can only choose 5 filters a minute. Sorry bruh :(( \n Now wait for 30 seconds";

          setTimeout(() => {
            this.isLoading = false;
            this.errorMessage = "";
          }, 30000);
        }
      }
    },
    sortCryptos(value) {
      switch (value) {
        case "Price":
          for (let i = 1; i < this.cryptocurrencies[0].length; i++) {
            if (
              this.cryptocurrencies[0][i].price >
              this.cryptocurrencies[0][i - 1].price
            ) {
              let temporaryLet = this.cryptocurrencies[0][i - 1];
              this.cryptocurrencies[0][i - 1] = this.cryptocurrencies[0][i];
              this.cryptocurrencies[0][i] = temporaryLet;
              i = 0;
            }
          }
          break;
        case "Risers":
          for (let i = 1; i < this.cryptocurrencies[0].length; i++) {
            if (
              this.cryptocurrencies[0][i].change >
              this.cryptocurrencies[0][i - 1].change
            ) {
              let temporaryLet = this.cryptocurrencies[0][i - 1];
              this.cryptocurrencies[0][i - 1] = this.cryptocurrencies[0][i];
              this.cryptocurrencies[0][i] = temporaryLet;
              i = 0;
            }
          }
          break;
        case "Failers":
          for (let i = 1; i < this.cryptocurrencies[0].length; i++) {
            if (
              this.cryptocurrencies[0][i].change <
              this.cryptocurrencies[0][i - 1].change
            ) {
              let temporaryLet = this.cryptocurrencies[0][i - 1];
              this.cryptocurrencies[0][i - 1] = this.cryptocurrencies[0][i];
              this.cryptocurrencies[0][i] = temporaryLet;
              i = 0;
            }
          }
          break;
        case "Market Cap":
          for (let i = 1; i < this.cryptocurrencies[0].length; i++) {
            if (
              this.cryptocurrencies[0][i].market_cap >
              this.cryptocurrencies[0][i - 1].market_cap
            ) {
              let temporaryLet = this.cryptocurrencies[0][i - 1];
              this.cryptocurrencies[0][i - 1] = this.cryptocurrencies[0][i];
              this.cryptocurrencies[0][i] = temporaryLet;
              i = 0;
            }
          }
          break;
      }
      this.mainKey++;
    },
  },
  computed: {
    filtersData() {
      // data about buttons
      return [
        [
          {
            text: "USD",
            key: "USD",
            active: true,
          },
          {
            text: "EUR",
            key: "EUR",
            active: false,
          },
          {
            text: "CZK",
            key: "CZK",
            active: false,
          },
        ],
        [
          {
            text: "Price",
            key: "Price",
            active: true,
          },
          {
            text: "Top Risers",
            key: "Top Risers",
            active: false,
          },
          {
            text: "Top Failers",
            key: "Top Failers",
            active: false,
          },
          {
            text: "Market Cap",
            key: "Market Cap",
            active: false,
          },
        ],
        [
          {
            text: "24h",
            key: "24h",
            active: true,
          },
          {
            text: "7d",
            key: "7d",
            active: false,
          },
          {
            text: "30d",
            key: "30d",
            active: false,
          },
          {
            text: "90d",
            key: "90d",
            active: false,
          },
          {
            text: "180d",
            key: "180d",
            active: false,
          },
          {
            text: "1y",
            key: "1y",
            active: false,
          },
        ],
      ];
    },
    cryptocurrencies() {
      // data about each cryptocurrency
      return [
        [
          {
            id: 1,
            name: "BitCoin",
            keyword: "bitcoin",
            imageLink:
              "https://assets.coingecko.com/coins/images/1/small/bitcoin.png?1547033579",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 2,
            name: "Ethereum",
            keyword: "ethereum",
            imageLink:
              "https://assets.coingecko.com/coins/images/279/small/ethereum.png?1595348880",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 3,
            name: "Ripple",
            keyword: "ripple",
            imageLink:
              "https://assets.coingecko.com/coins/images/44/small/xrp-symbol-white-128.png?1605778731",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 4,
            name: "Polkadot",
            keyword: "polkadot",
            imageLink:
              "https://assets.coingecko.com/coins/images/12171/small/polkadot.png?1639712644",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 5,
            name: "Cardano",
            keyword: "cardano",
            imageLink:
              "https://assets.coingecko.com/coins/images/975/small/cardano.png?1547034860",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 6,
            name: "Chainlink",
            keyword: "chainlink",
            imageLink:
              "https://assets.coingecko.com/coins/images/877/small/chainlink-new-logo.png?1547034700",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 7,
            name: "Dogecoin",
            keyword: "dogecoin",
            imageLink:
              "https://assets.coingecko.com/coins/images/5/small/dogecoin.png?1547792256",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 8,
            name: "Stellar",
            keyword: "stellar",
            imageLink:
              "https://assets.coingecko.com/coins/images/100/small/Stellar_symbol_black_RGB.png?1552356157",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 9,
            name: "USD Coin",
            keyword: "usd-coin",
            imageLink:
              "https://assets.coingecko.com/coins/images/6319/small/USD_Coin_icon.png?1547042389",
            price: 0,
            change: 0,
            market_cap: 0,
          },
          {
            id: 10,
            name: "Solana",
            keyword: "solana",
            imageLink:
              "https://assets.coingecko.com/coins/images/4128/small/solana.png?1640133422",
            price: 0,
            change: 0,
            market_cap: 0,
          },
        ],
      ];
    },
  },
};
</script>

<style>
#app {
  color: #f9f9ed;
  font-family: "Heebo", sans-serif;
  box-sizing: border-box;
}
</style>
