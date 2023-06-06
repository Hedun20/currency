<template>
  <div class="currency-converter">
    <h2 class="title">Currency Converter</h2>
    <form @submit.prevent="convertCurrency" class="form">
      <div class="form-group">
        <label for="amount">Amount:</label>
        <input
          type="number"
          id="amount"
          v-model="amount"
          required
          class="form-input"
        />
      </div>
      <div class="form-group">
        <label for="from">From:</label>
        <select id="from" v-model="fromCurrency" required class="form-select">
          <option value="UAH">UAH</option>
          <option value="EUR">EUR</option>
          <option value="USD">USD</option>
        </select>
      </div>
      <div class="form-group">
        <label for="to">To:</label>
        <select id="to" v-model="toCurrency" required class="form-select">
          <option value="UAH">UAH</option>
          <option value="EUR">EUR</option>
          <option value="USD">USD</option>
        </select>
      </div>
      <button type="submit" class="form-button">Convert</button>
    </form>
    <div v-if="result" class="result">
      <p>{{ amount }} {{ fromCurrency }} = {{ result }} {{ toCurrency }}</p>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      amount: 0,
      fromCurrency: "",
      toCurrency: "",
      result: null,
    };
  },
  methods: {
    async convertCurrency() {
      try {
        const response = await axios.get(
          "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json"
        );
        const exchangeRates = response.data;

        const fromCurrencyRate =
          this.fromCurrency === "UAH"
            ? 1
            : exchangeRates.find((rate) => rate.cc === this.fromCurrency)?.rate;
        const toCurrencyRate =
          this.toCurrency === "UAH"
            ? 1
            : exchangeRates.find((rate) => rate.cc === this.toCurrency)?.rate;

        if (fromCurrencyRate && toCurrencyRate) {
          let result;
          if (this.fromCurrency === "UAH" && this.toCurrency !== "UAH") {
            result = this.amount / toCurrencyRate;
          } else if (this.fromCurrency !== "UAH" && this.toCurrency === "UAH") {
            result = this.amount * fromCurrencyRate;
          } else if (this.fromCurrency === "USD" && this.toCurrency === "EUR") {
            result = this.amount * (fromCurrencyRate / toCurrencyRate);
          } else if (this.fromCurrency === "EUR" && this.toCurrency === "USD") {
            result = this.amount * (fromCurrencyRate / toCurrencyRate);
          } else if (this.fromCurrency === "USD" && this.toCurrency === "UAH") {
            result = this.amount * fromCurrencyRate;
          } else if (this.fromCurrency === "UAH" && this.toCurrency === "USD") {
            result = this.amount / fromCurrencyRate;
          } else if (this.fromCurrency === "EUR" && this.toCurrency === "UAH") {
            result = this.amount * fromCurrencyRate;
          } else if (this.fromCurrency === "UAH" && this.toCurrency === "EUR") {
            result = this.amount / fromCurrencyRate;
          } else {
            result = this.amount;
          }

          this.result = result.toFixed(2);
        } else {
          throw new Error("Invalid currency selection");
        }
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>

<style scoped>
.currency-converter {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f5f5f5;
  border: 1px solid #ddd;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.title {
  font-weight: 600;
  font-size: 24px;
  margin-bottom: 20px;
  text-align: center;
  color: #428bca;
}

.form {
  font-weight: 400;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  font-weight: bold;
  color: #428bca;
}

.form-input {
  font-size: 16px;
  width: 85%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 3px;
}

.form-select {
  font-size: 16px;
  width: 90%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 3px;
}

.form-button {
  font-size: 16px;
  width: 90%;
  padding: 10px;
  background-color: #428bca;
  color: #fff;
}

.result {
  padding: 10px;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 3px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}

.result p {
  margin: 0;
  font-weight: bold;
  color: #428bca;
}
</style>
