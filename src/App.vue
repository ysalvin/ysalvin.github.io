<template>
    <header>
        <!-- {{ firstName }} {{lastName}}  -->
        <h1>Currency Exchange App</h1>
    </header>
    <main>

        <div class="currency_one">
          <h3>From</h3>
          <select name="currency1" id="currency1" v-model= "currency_one" @change="changing_currrency">
            <option v-for="(value, key) in currency_list" :value="key">
              {{key}} - {{value}}
            </option>
          </select>
          <input class="inputbox" type="number" name="currency1_amount" v-model=value_one id="currency1_amount" min="0" placeholder="Type here" @change="calculate"/>
        </div>

        <form>
            <input class="checkbox" type="checkbox" v-model="past" @change="get_historical"> <label>Historical Data</label><br>
            <input type="date" v-model="past_date" id="date_box" pattern="\d{4}-\d{2}-\d{2}"  @change="get_historical" style="color:blue;text-align:center"/>
        </form>

        <div class="switch">
          <button class="switch_button" @click="switch_button"> Switch</button>
          <span>  1 {{this.currency_one}} = {{this.base_rate}} {{this.currency_two}}</span>
        </div>

        <div class="currency_two">
          <h3>To</h3>
          <select name="currency2" id="currency2" v-model= "currency_two" @change="changing_currrency">
            <option v-for="(value, key) in currency_list" :value="key">
              {{key}} - {{value}}
            </option>
          </select>
          <input class="inputbox" type="number" name="currency2_amount" v-model=value_two id="currency2_amount" placeholder="Exchaged amount" readonly/>
        </div>

        <div>
          quota_used {{quota_used}} <br>
          quota_remaining {{quota_remaining}}<br>
          last_update {{last_update}}
        </div> 

    </main>
</template>

<script >
import $ from 'jquery'
export default {
  data() {
    return {
      past: false,
      past_date:"",
      currency_list:[],
      currency_data:[],
      currency_base:"",
      currency_one:"USD",
      currency_two:"HKD",
      base_rate: 0,
      value_one: 1,
      value_two: 0,
      quota_used: 0,
      quota_remaining: 0,
      last_update: "",
    };
  },
  methods:{

    fetch_currency_list(){
      $.get('https://openexchangerates.org/api/currencies.json')
        .then((response) =>{
          this.currency_list = response;
        })
        .catch((error) =>{
          console.log(error);
        })
    },

    fetch_usage(){
      $.get('https://openexchangerates.org/api/usage.json', {app_id: '8e51b66efd3a4aaa9bc2b7585804b4fd'})
        .then((response) =>{
          // console.log(response.data);
          this.quota_used = response.data.usage.requests;
          this.quota_remaining = response.data.usage.requests_remaining;
        })
        .catch((error) =>{
          console.log(error);
        })
    },

    fetch_rate(){
      $.get('https://openexchangerates.org/api/latest.json', {app_id: '8e51b66efd3a4aaa9bc2b7585804b4fd'}).then((data) => {
        console.log(data);
        this.currency_base = data.base;
        this.currency_data = data.rates;
        this.changing_currrency();
        this.last_update = new Date();

        this.calculate();
      }).catch((error) =>{
          console.log(error);
        })
    },

    fetch_historical_data(){
      $.get('https://openexchangerates.org/api/historical/' + this.past_date + '.json', {app_id: '8e51b66efd3a4aaa9bc2b7585804b4fd'}) .then((data) => {
        this.currency_data=data.rates;
        console.log(this.currency_data);
        this.changing_currrency();
        this.calculate();
      }).catch((error) =>{
          console.log(error);
      });
    },

    get_historical(event){
      if(this.past){
        this.fetch_historical_data();
      }
      else{
        this.fetch_rate();
        this.changing_currrency();
        this.calculate();
      }
    },

    changing_currrency(){
      // console.log(this.currency_data);
      // console.log(this.currency_base);

      let r1 = this.currency_data[this.currency_one];
      let r2 = this.currency_data[this.currency_two];
      // console.log(r1, r2);
      this.base_rate=1/r1*r2;
      this.calculate();
    },

    switch_button(event){
      console.log("switch");
      var swap = this.currency_one;
      this.currency_one = this.currency_two;
      this.currency_two =swap;
      swap = this.value_one;
      this.value_one = this.value_two;
      this.value_two = swap;

      this.changing_currrency();
  },

    calculate(event){
      let r1 = this.currency_data[this.currency_one];
      let r2 = this.currency_data[this.currency_two];
      if (r1 == 0){
        alert("this currency is not supported")
      }
      this.value_two=this.value_one/r1*r2;
    },

  },

  

  mounted(){
    this.fetch_currency_list();
    this.fetch_usage();
    this.fetch_rate();


    //setting default date for past_date
    var date = new Date();

    var day = date.getDate();
    var month = date.getMonth() + 1;
    var year = date.getFullYear();

    if (month < 10) month = "0" + month;
    if (day < 10) day = "0" + day;

    var today = year + "-" + month + "-" + day ; 
    this.past_date = today;
  }
}
</script>


<style > 
  html{
    background-color: rgba(128, 133, 122, 0.28);
  }
  #app{
    display:flex;
    flex-direction: column;
    align-items: center;
    align-content: center;
    width: 100%;
    height: 100%;
    font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }
  h1{
    font-size: 30px;
  }
  .currency_one,.currency_two{
    padding: 10px;
    width: 500px;
    height: 170px; 
    background-color: yellow;
    border-radius: 25px;
  }

  select{
    padding: 5px;
    margin: 5px;
    border: 2px solid rgba(0,0,0,0.2);
    border-radius: 10px;
    font-size: 14px;
  }

  .inputbox{
    padding: 5px;
    margin: 5px;
    border: 2px solid rgba(128,128,128,0.5);
    border-radius: 10px;
    height: 20px;
    width: 200px;
    font-size: 20px;
  }

  .switch_button{
    margin-top: 30px;
    margin-right: 30px;
    margin-bottom: 30px;
    background-color: rgba(0, 255, 0, 0.5);
  }

  .switch{
    display: inline-block;
  }

  form{
    padding: 5px;
    margin: 5px;
    background-color: aquamarine;
    border-radius: 20px;
    width: 500px;
    display: block;
    text-align: center;
  }

  
  
</style>
