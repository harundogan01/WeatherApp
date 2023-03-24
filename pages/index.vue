<template>
  <v-card class="card-container">
    <div class="col-xs-10 col-sm-8 col-md-5 col-lg-5 col-xs-5 ma-auto justify-center align-center text-center">
<v-autocomplete
v-model="selectedItem"
clearable
:loading="isLoading"
:items="search_results"
item-text="name"
item-value="id"
@keydown.enter="getSearchedData($event.target.value)"
:search-input.sync="search"
:getSelectedData="getSelectedData"
label="Specific Location" 
    placeholder="Enter the place where you want to access the location information" 
>
</v-autocomplete>
<v-alert v-if="this.selectedItem === null || this.selectedItem === ''" type="info" color="blue">
Enter the name of the region (for example, New York) where you want to see the weather information and press Enter.</v-alert>
    </div>
    <v-card-title v-if="(items_one.length > 0 && items_two.length > 0) || (search_results.length > 0 && this.selectedItem !== null)" class="text-center justify-center">
      <p>{{ area !== city ? area + ',' + city + ',' + country : city + ',' + country }} Current Weather</p>
    </v-card-title>
      <p class="text-center justify-center" v-if="(items_one.length > 0 && items_two.length > 0) || (search_results.length > 0 && this.selectedItem !== null)">
        Last Update Time: {{ last_update_time }}</p>
    <v-tabs centered v-if="(items_one.length > 0 && items_two.length > 0) || (search_results.length > 0 && this.selectedItem !== null)">
      <v-tab>Current Weather</v-tab>
      <v-tab>Three Day Weather Forecast</v-tab>
      <v-tab-item>
        <v-card-text>
      <v-list class="text-center">
  <v-list-item-group
        class="leftBlock d-inline-block col-xs-6 col-sm-6 col-md-5 col-lg-5 col-xl-6"
    color="grey"
  >
    <v-list-item
      v-for="(item, i) in items_one"
      :key="i"
    >
      <v-list-item-icon v-if="item.title !== 'Weather Status'">
        <v-icon v-text="item.icon"></v-icon>
      </v-list-item-icon>
      <v-list-item-avatar v-else>
        <v-img :src="item.icon"></v-img>
      </v-list-item-avatar>
      <v-list-item-content>
        <v-list-item-title v-text="item.title + ' : ' + item.text"></v-list-item-title>
      </v-list-item-content>
    </v-list-item>
  </v-list-item-group>
  <v-list-item-group
    color="grey"
    class="centerBlock d-inline-block col-xs-6 col-sm-6 col-md-5 col-lg-5 col-xl-6"
  >
    <v-list-item
      v-for="(item, i) in items_two"
      :key="i"
    >
      <v-list-item-icon>
        <v-icon v-text="item.icon"></v-icon>
      </v-list-item-icon>
      <v-list-item-content>
        <v-list-item-title v-text="item.title + ' : ' + item.text"></v-list-item-title>
      </v-list-item-content>
    </v-list-item>
  </v-list-item-group>
</v-list>
      </v-card-text>
    </v-tab-item>
      <v-tab-item>
        <div class="d-flex justify-center align-center text-center">
            <v-card
class="ml-15 mr-15"
max-width="800"
v-for="(item, i) in forecast" :key="i"
>
<v-list-item two-line>
  <v-list-item-content>
    <v-list-item-subtitle>{{ item.date }}, {{ item.day.condition.text }}</v-list-item-subtitle>
  </v-list-item-content>
</v-list-item>

<v-card-text>
  <v-row align="center">
    <v-col
      class="text-h2"
      cols="6"
      title="Average Temperature"
    >
      {{ item.day.avgtemp_c }}&deg;C
    </v-col>
    <v-col cols="6">
      <v-img
        :src="item.day.condition.icon"
        :alt="item.day.condition.text"
        width="92"
      ></v-img>
    </v-col>
  </v-row>
</v-card-text>

<v-list-item>
  <v-list-item-icon>
    <v-icon>mdi-send</v-icon>
  </v-list-item-icon>
  <v-list-item-subtitle>{{ item.day.maxwind_kph }} km/h</v-list-item-subtitle>
</v-list-item>

<v-list-item>
  <v-list-item-icon>
    <v-icon>mdi-waves-arrow-up</v-icon>
  </v-list-item-icon>
  <v-list-item-subtitle title="Average Humidity">{{ item.day.avghumidity }}%</v-list-item-subtitle>
</v-list-item>
<v-list-item>
  <v-list-item-icon>
    <v-icon>mdi-thermometer-minus</v-icon>
  </v-list-item-icon>
  <v-list-item-subtitle title="Minimum Temperature">{{ item.day.mintemp_c }}%</v-list-item-subtitle>
</v-list-item>
<v-list-item>
  <v-list-item-icon>
    <v-icon>mdi-thermometer-plus</v-icon>
  </v-list-item-icon>
  <v-list-item-subtitle title="Maximum Temperature">{{ item.day.maxtemp_c }}%</v-list-item-subtitle>
</v-list-item>
</v-card>
</div>
      </v-tab-item>
    </v-tabs>
  </v-card>
</template>

<script>
import moment from 'moment'
import VueGeolocation from 'vue-browser-geolocation';
import axios from 'axios';
export default {
name: 'IndexPage',
data() {
return {
  model: null,
  search: null,
  isLoading: false,
  last_update_time: null,
  localTime: null,
  area: null,
  weatherImage: null,
  weatherStatus: null,
  city: null,
  country: null,
  searchedLocation: "",
  celsius: null,
  items_one: [],
  items_two: [],
  forecast: [],
  search_results: [],
  selectedItem: null,
  latAndLonBySearchedLocation: null
}
},
created() {
this.getUserLocation();
},
computed: {
getSelectedData() {
  var res = this.search_results.filter((val) => val.id === this.selectedItem)
  res.forEach((val) => {
    this.latAndLonBySearchedLocation = val.lat + ',' + val.lon;
    this.getUserLocation(this.latAndLonBySearchedLocation);
  })
}
},
methods: {
getSearchedData(val) {
  if (val === null || val === "")
    this.search_results = [];
  else {
    this.isLoading = true;
  axios.get(process.env.baseUrl + 'search.json', {
      headers : {
        'Content-Type' : 'application/x-www-form-urlencoded; charset=UTF-8'
      },
      params: {
        key: "77fafc516136415a8ee122058232203",
        q: val
      }
    })
      .then((res) => {
        if (res.status) {
          res.data.forEach((val) => {
            this.search_results.push( { name: val.name, region: val.region, country: val.country, id: val.id, lat: val.lat, lon: val.lon });
          })
          this.isLoading = false;
        }
      })
      .catch(err => {
        console.log(err)
      })
      .finally(() => (this.isLoading = false))
  }
},
getUserLocation(param) {
  if (param === null || param === "" || param === undefined) {
    VueGeolocation.getLocation()
.then(coordinates => {
    axios.get(process.env.baseUrl + 'forecast.json', {
      headers : {
        'Content-Type' : 'application/x-www-form-urlencoded; charset=UTF-8'
      },
      params: {
        key: "77fafc516136415a8ee122058232203",
        q: coordinates.lat + ',' + coordinates.lng,
        days: 4
      }
    })
    .then((resp) => {
      resp.data.forecast.forecastday.forEach((val, index) => {
        if (index > 0) {
          this.forecast.push(val);
        }
      })
        this.items_one.push(
        { title: "Weather Status", text: resp.data.current.condition.text, icon: resp.data.current.condition.icon},
        { title: "Humidity", text: resp.data.current.humidity, icon: "mdi-waves-arrow-up" },
        { title: "Latitude", text: resp.data.location.lat, icon: "mdi-latitude" },
        { title: "Longitude", text: resp.data.location.lon, icon: "mdi-longitude" },
        { title: "Wind Degree", text: resp.data.current.wind_degree, icon: "mdi-wind-power" },
        { title: "Pressure in milibars", text: resp.data.current.pressure_mb, icon: "mdi-car-brake-low-pressure" },
        { title: "Wind speed per hour(km)", text: resp.data.current.wind_kph, icon: "mdi-wind-power" },
        { title: "Gust in kilometer per hour", text: resp.data.current.gust_kph, icon: "mdi-weather-windy" },
      )
      this.items_two.push(
        { title: "Temperature in Celsius", text: resp.data.current.temp_c, icon: "mdi-temperature-celsius"},
        { title: "Temperature in Fahrenheit", text: resp.data.current.temp_f, icon: "mdi-temperature-fahrenheit" },
        { title: "Felt temperature", text: resp.data.current.feelslike_c, icon: "mdi-temperature-celsius" },
        { title: "Felt temperature", text: resp.data.current.feelslike_f, icon: "mdi-temperature-fahrenheit" },
        { title: "Wind Direction", text: resp.data.current.wind_degree, icon: "mdi-wind-power-outline" },
        { title: "Precipitation amount in (mm)", text: resp.data.current.precip_mm, icon: "mdi-weather-pouring" },
        { title: "Wind speed per hour(mi)", text: resp.data.current.wind_mph, icon: "mdi-wind-power" },
        { title: "Gust in miles per hour", text: resp.data.current.gust_mph, icon: "mdi-weather-windy" },
      )
      this.last_update_time = moment(resp.data.current.last_updated).format("DD-MM-YYYY LT");
      this.area = resp.data.location.name;
      this.city = resp.data.location.region;
      this.country = resp.data.location.country;
      this.weatherImage = resp.data.current.condition.icon;
      this.weatherStatus = resp.data.current.condition.text;
    })
});
  }
  else {
    this.forecast = [];
    this.items_one = [];
    this.items_two = [];
    axios.get(process.env.baseUrl + 'forecast.json', {
      headers : {
        'Content-Type' : 'application/x-www-form-urlencoded; charset=UTF-8'
      },
      params: {
        key: "77fafc516136415a8ee122058232203",
        q: param,
        days: 4
      }
    })
    .then((resp) => {
      resp.data.forecast.forecastday.forEach((val, index) => {
        if (index > 0) {
          this.forecast.push(val);
        }
      })
        this.items_one.push(
        { title: "Weather Status", text: resp.data.current.condition.text, icon: resp.data.current.condition.icon},
        { title: "Humidity", text: resp.data.current.humidity, icon: "mdi-waves-arrow-up" },
        { title: "Latitude", text: resp.data.location.lat, icon: "mdi-latitude" },
        { title: "Longitude", text: resp.data.location.lon, icon: "mdi-longitude" },
        { title: "Wind Degree", text: resp.data.current.wind_degree, icon: "mdi-wind-power" },
        { title: "Pressure in milibars", text: resp.data.current.pressure_mb, icon: "mdi-car-brake-low-pressure" },
        { title: "Wind speed per hour(km)", text: resp.data.current.wind_kph, icon: "mdi-wind-power" },
        { title: "Gust in kilometer per hour", text: resp.data.current.gust_kph, icon: "mdi-weather-windy" },
      )
      this.items_two.push(
        { title: "Temperature in Celsius", text: resp.data.current.temp_c, icon: "mdi-temperature-celsius"},
        { title: "Temperature in Fahrenheit", text: resp.data.current.temp_f, icon: "mdi-temperature-fahrenheit" },
        { title: "Felt temperature", text: resp.data.current.feelslike_c, icon: "mdi-temperature-celsius" },
        { title: "Felt temperature", text: resp.data.current.feelslike_f, icon: "mdi-temperature-fahrenheit" },
        { title: "Wind Direction", text: resp.data.current.wind_degree, icon: "mdi-wind-power-outline" },
        { title: "Precipitation amount in (mm)", text: resp.data.current.precip_mm, icon: "mdi-weather-pouring" },
        { title: "Wind speed per hour(mi)", text: resp.data.current.wind_mph, icon: "mdi-wind-power" },
        { title: "Gust in miles per hour", text: resp.data.current.gust_mph, icon: "mdi-weather-windy" },
      )
      this.last_update_time = moment(resp.data.current.last_updated).format("DD-MM-YYYY LT");
      this.area = resp.data.location.name;
      this.city = resp.data.location.region;
      this.country = resp.data.location.country;
      this.weatherImage = resp.data.current.condition.icon;
      this.weatherStatus = resp.data.current.condition.text;
    })
  }
},
},
watch: {
search(val) {
  if (val === null || val === "")
    this.search_results = [];
}
}
}
</script>