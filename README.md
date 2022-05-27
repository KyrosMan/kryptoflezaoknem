# hello-world

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


This project uses CoinGecko API - https://www.coingecko.com/en/api/documentation to get data about cryptocurrencies
First container contains filter buttons. The second one has 10 components with the 10 most popular cryptocurrencies

Computed
I store all information about filter buttons and cryptocurrencies in the computed
I use the arrays to v-for loop create all the components

Each time a filter is clicked (except for Prices, Top Risers and Top Failers) I send a request for each cryptocurrency to get their price.
The free version of the API only gives permission for 50 calls a minute, so if the user clicks too many filters too fast a loading screen for 30 seconds will appear.
