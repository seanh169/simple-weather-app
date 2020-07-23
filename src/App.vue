

<template>
  <div id="app">

      <b-navbar type="dark" variant="dark"> 
            <img src="./assets/iconfinder_07_sun_smile_happy_emoticon_weather_smiley_3375693.svg" width="50" height="50" />
            <b-navbar-brand href="#" style="margin-left: 25px">{{appBarTitle}}</b-navbar-brand>
      </b-navbar>

      

      <b-container id="container">
        <b-form v-on:submit="onSubmit" v-on:reset="onReset" v-if="show">
          <b-row class="mb-3">
            <b-col md="5" offset-md="4" align-self="center">
              <small>Please enter city name or press button below to insert longitute/latitude to find a city's five day forecast.</small>
            </b-col>
          </b-row>

          <b-row class="mb-3">
            <b-col md="5" offset-md="4" align-self="center">
              <b-button variant="primary" v-on:click="changeLocationType">Click here to search by {{showCityType ? "Longitude/Latitude" : "City Name"}}</b-button>
              <small></small>
            </b-col>
          </b-row>


          <b-row v-if="showCityType" class="mb-3">
            <b-col md="5" offset-md="4" align-self="center">
            <b-input-group size="md" prepend="City Name">
              <b-form-input type="text" v-model="cityName"></b-form-input>
            </b-input-group>
            </b-col>
          </b-row>

          <div v-else>
          <b-row class="mb-3">
            <b-col md="5" offset-md="4" align-self="center">
            <b-input-group size="md" prepend="Longitude">
              <b-form-input type="number" v-model="longitute"></b-form-input>
            </b-input-group>
            </b-col>
          </b-row>

          <b-row class="mb-3">
            <b-col md="5" offset-md="4" align-self="center">
            <b-input-group size="md" prepend="Latitude">
              <b-form-input type="number" v-model="latitude"></b-form-input>
            </b-input-group>
            </b-col>
          </b-row>
          </div>
          <b-row class="mb-3">
          <b-col md="5" offset-md="5" align-self="center">
            <b-button type="submit" variant="success">Search</b-button>
            <b-button type="reset" variant="danger">Reset</b-button>
          </b-col>
          </b-row>

          <b-row class="mb-3">
          <b-col md="5" offset-md="4" align-self="center">
          <b-alert dismissible variant="danger" fade :show="showDismissibleAlert" @dismissed="showDismissibleAlert=false" v-for="(error, index) in errors" :key="index">
            {{error}}
          </b-alert>
          </b-col>
          </b-row>
        </b-form>

        <div v-if="showListOfCities">
        <b-row class="mb-3">
          <b-col md="5" offset-md="4" align-self="center">
           <b-form-group label="Choose city below">
            <b-form-radio v-model="selectedCity" @change="radioChangeHandler(city.title)" name="city-radios" v-for="city in listOfCities" :key="city.woeid" :value="city.woeid" >{{city.title}}</b-form-radio>
          </b-form-group>
          </b-col>
        </b-row>

        <b-row class="mb-3">
          <b-col md="5" offset-md="4" align-self="center">
            <b-button variant="success" v-on:click="showForeCastHandler">Show 5 Day Forecast!</b-button>
          </b-col>
        </b-row>
        </div>

        <div v-if="showfiveDayForeCast" >
        <b-row class="mb-3">
          <b-col md="5" offset-md="4" align-self="center">
            <h6>Five Day Forecast</h6>
            <b-card
              v-for="(day, index) in fiveDayValues"
              :key="index"
              :title="day.date"
              img-top
              tag="article"
              style="max-width: 25rem;"
              class="mb-2"
            >
              <b-card-text>
                  <b-list-group>
                    <b-list-group-item>Location : {{validatedCityName}}</b-list-group-item>
                    <b-list-group-item>
                      Weather state : {{day.weather_state}} <img :src="getSvgIcon(day.weather_state)" alt="Weather icon" width="25" height="25">
                    </b-list-group-item>
                    <b-list-group-item>High temperature : {{day.high_temp.toFixed(2)}} &#8451;</b-list-group-item>
                    <b-list-group-item>Low temperature : {{day.low_temp.toFixed(2)}} &#8451;</b-list-group-item>
                    <b-list-group-item>Wind speed : {{day.wind_speed.toFixed(2)}} mph</b-list-group-item>
                    <b-list-group-item>Wind direction : {{day.wind_direction.toFixed(2)}} </b-list-group-item>
                    <b-list-group-item>Air Pressure: {{day.air_pressure.toFixed(2)}} psi</b-list-group-item>
                    <b-list-group-item>Humidity : {{day.humidity.toFixed(2)}} &#37;</b-list-group-item>
                  </b-list-group>
               </b-card-text>
            </b-card>
          </b-col>
        </b-row>
        </div>

      </b-container>

      <b-modal ref="modal-1" id="modal-1" title="Weather App" ok-only ok-title="Cancel">

        <p class="md-4">Searching. Please wait...</p>
        <div class="d-flex justify-content-center mb-3">
        <b-spinner variant="primary" type="grow" label="Spinning"></b-spinner>
        </div>
      </b-modal>

  </div>
</template>

<script>
import axios from 'axios';
var config = {
    headers: {'Access-Control-Allow-Origin': '*'}
};
export default {
  name: 'app',
  data () {
    return {
      appBarTitle: 'Weather App',
      buttonText: 'By City',
      showCityType: true,
      cityName: null,
      longitute: null,
      validatedCityName: '',
      latitude: null,
      show: true,
      errors: [],
      listOfCities: [],
      showDismissibleAlert: false,
      showListOfCities: false,
      selectedCity: '',
      fiveDayValues: [],
      showfiveDayForeCast: false
    }
  },
  methods : {
    changeLocationType: function(event) {
        
      this.showCityType = !this.showCityType;
      
    },
    onSubmit: function(event) {
      event.preventDefault();
      var vm = this;
      vm.showfiveDayForeCast = false;
      this.errors = [];

      if(this.showCityType && !this.cityName) {
        this.errors.push("City name is required");
        this.showDismissibleAlert = true;
        return false;
      }  
      if(!this.showCityType && (!this.longitute || !this.latitude)) {
        this.errors.push("Longitude and Latitude are required");
        this.showDismissibleAlert = true;
        return false;
      }  
      
      if(this.showCityType){
        vm.$refs['modal-1'].show();
        axios.get("https://cors-anywhere.herokuapp.com/https://www.metaweather.com/api/location/search/?query=" + this.cityName)
        .then(res=>{
          if(res.data.length < 1){
            this.errors.push("There are no city results with that name. Please Try again.");
            this.showDismissibleAlert = true;
          }
          if(res.data.length > 0){
            let returnArr = [];
            this.listOfCities = res.data;
            this.showListOfCities = true;
          }
          vm.$refs['modal-1'].hide()
        })
        .catch(err=>{
          this.errors.push("Error retrieving data.");
          this.showDismissibleAlert = true;
          vm.$refs['modal-1'].hide()
        })

      }

      if(!this.showCityType){
        vm.$refs['modal-1'].show();
        axios.get("https://cors-anywhere.herokuapp.com/https://www.metaweather.com/api/location/search/?lattlong=" + this.latitude + "," + this.longitute + "")
        .then(res=>{
          if(res.data.length < 1){
            this.errors.push("There are no city results with that name. Please Try again.");
            this.showDismissibleAlert = true;
          }
          if(res.data.length > 0){
            let returnArr = [];
            this.listOfCities = res.data;
            this.showListOfCities = true;
          }
          vm.$refs['modal-1'].hide()
        })
        .catch(err=>{
          this.errors.push("Error retrieving data.");
          this.showDismissibleAlert = true;
          vm.$refs['modal-1'].hide()
        })
      }
      
      
    },

    showForeCastHandler: function(event) {
      var vm = this;
      vm.$refs['modal-1'].show()
      const date = new Date();
      let thisYear = date.getFullYear();
      let today = date.getDate();
      let month = date.getMonth();
      let dayOne = axios.get("https://cors-anywhere.herokuapp.com/https://www.metaweather.com/api/location/" + vm.selectedCity + "/" + thisYear + "/" + month + "/" + today + "/");
      let dayTwo = axios.get("https://cors-anywhere.herokuapp.com/https://www.metaweather.com/api/location/" + vm.selectedCity + "/" + thisYear + "/" + month + "/" + (today + 1) + "/");
      let dayThree = axios.get("https://cors-anywhere.herokuapp.com/https://www.metaweather.com/api/location/" + vm.selectedCity + "/" + thisYear + "/" + month + "/" + (today + 2) + "/");
      let dayFour = axios.get("https://cors-anywhere.herokuapp.com/https://www.metaweather.com/api/location/" + vm.selectedCity + "/" + thisYear + "/" + month + "/" + (today + 3) + "/");
      let dayFive = axios.get("https://cors-anywhere.herokuapp.com/https://www.metaweather.com/api/location/" + vm.selectedCity + "/" + thisYear + "/" + month + "/" + (today + 4) + "/");

      Promise.all([dayOne, dayTwo, dayThree, dayFour, dayFive])
        .then(res=>{
          res.map(val=>{
             this.fiveDayValues.push({
                date: val.data[0].applicable_date,
                cityName: this.validatedCityName,
                weather_state: val.data[0].weather_state_name,
                high_temp: val.data[0].max_temp,
                low_temp: val.data[0].min_temp,
                wind_speed: val.data[0].wind_speed,
                air_pressure: val.data[0].air_pressure,
                humidity: val.data[0].humidity,
                wind_direction: val.data[0].wind_direction

             })
          })
          vm.showListOfCities = false;
          vm.showfiveDayForeCast = true;
          vm.$refs['modal-1'].hide()
        })
        .catch(err=>{
          this.errors.push("Error retrieving data.");
          this.showDismissibleAlert = true;
          vm.$refs['modal-1'].hide()
      })
        .catch(err=>{
          this.errors.push("Error retrieving data.");
          this.showDismissibleAlert = true;
          vm.$refs['modal-1'].hide()
        })
        
       
     
    }, 
    onReset: function(event) {
      this.showListOfCities = false;
      this.showfiveDayForeCast = false;
      this.showDismissibleAlert = false;
      this.showCityType = true;
      this.longitute = null;
      this.latitude = null;
      this.cityName = null;
      this.validatedCityName = '';
      
    },
    radioChangeHandler: function(event, other) {
      this.validatedCityName = event;
    },
    getSvgIcon : function(inputStr) {
      switch(inputStr){
        case "Snow":
          return "https://www.metaweather.com/static/img/weather/sn.svg"; 
          break; 
        case "Sleet":
          return "https://www.metaweather.com/static/img/weather/sl.svg";  
          break;
        case "Hail":
          return "https://www.metaweather.com/static/img/weather/h.svg";  
          break;
        case "Thunderstorm": 
          return "https://www.metaweather.com/static/img/weather/t.svg"; 
          break;
        case "Heavy Rain":
          return "https://www.metaweather.com/static/img/weather/hr.svg";  
          break; 
        case "Light Rain": 
          return "https://www.metaweather.com/static/img/weather/lr.svg";  
          break; 
        case "Showers": 
          return "https://www.metaweather.com/static/img/weather/s.svg";  
          break; 
        case "Heavy Cloud": 
          return "https://www.metaweather.com/static/img/weather/hc.svg";  
          break; 
        case "Light Cloud": 
          return "https://www.metaweather.com/static/img/weather/lc.svg"; 
          break; 
        case "Clear": 
          return "https://www.metaweather.com/static/img/weather/c.svg";  
          break;
      }
      return "";
    }

  },
  watch: {
    showCityType: function(){
      var vm = this;  
      vm.buttonText = this.showCityType ? 'By City' : 'By Long/Lat';
    }
  }
}
</script>

<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  
}
#container {
  padding-top: 20px;
}
</style>
