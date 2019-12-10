## config
export const baseImg = 'https://www.weatherbit.io/static/img/icons/'

export const initCity = {city: 'Madrid', country: 'Spain', key: 'es'}

export const cities =  [
	{city: 'Minsk', country: 'Belarussia', key: 'Bl'},
	{city: 'Gomel', country: 'Belarussia', key: 'Bl'},
	{city: 'Grodno', country: 'Belarussia', key: 'Bl'},
	{city: 'Mogiley', country: 'Belarussia', key: 'Bl'},
	{city: 'Vitebsk', country: 'Belarussia', key: 'Bl'}
	{city: 'Breast', country: 'Belarussia', key: 'Bl'}

##main
import Vue from 'vue'
import App from './App.vue'
import store from './store'

Vue.config.productionTip = false

new Vue({
  store,
  render: h => h(App)
}).$mount('#app')

## store
import Vue from 'vue'
import Vuex from 'vuex'
import AppServices from '@/services/AppServices'

Vue.use(Vuex)

const state = {  
  currentWeather: null,
  forecastWeather: [],
  city: '',
  country: '',
  scale: 'C'
}

const getters = {
  temp: state => (state.currentWeather != null) ? state.currentWeather.temp : null,
  datetime: state => (state.currentWeather != null) ? state.currentWeather.ob_time : '',
  weather: state =>  (state.currentWeather != null) ? state.currentWeather.weather : null
}

const mutations = {
  START_SEARCHING (state, value) {
    state.city = value.city;
    state.country = value.country;    
  },  
  UPDATE_WEATHER (state, { current, forecast }) {
    Vue.set(state, 'currentWeather', current)
    Vue.set(state, 'forecastWeather', forecast)
  },  
  UPDATE_SCALE (state, scale) {
    state.scale = scale
  },  
  FIRE_ERROR (state) {
    Vue.set(state, 'currentWeather', {})
    Vue.set(state, 'forecastWeather', [])
  }  
}

const actions = {
  SET_SCALE({ commit }, scale){
    commit('UPDATE_SCALE', scale);
  },
  async FETCH_WEATHER ({ commit }, value) {
    commit('START_SEARCHING', value);

    let current = null;
    let forecast = [];
 
    if (value.key != '') {
      try{
        const currentWeather = await AppServices.getCurrentWeather({
          city: value.city,
          country: value.key
        })
  
        current = (currentWeather.data != '') ? currentWeather.data.data[0] : null;
      }
      catch (e){
        current = null
      }
  
      try{    
        const forecastWeather = await AppServices.getForecastWeather({
          days: 5,
          city: value.city,
          country: value.key
        })
  
        forecast = (forecastWeather.data != '') ? forecastWeather.data.data : [];
      }
      catch (e){
        forecast = []
      }
    }

    commit('UPDATE_WEATHER', { current, forecast })  
  }
}

const store = new Vuex.Store({
  state,
  mutations,
  actions,
  getters
})

export default store

## services/api

import axios from "axios";

export const API = axios.create({
  baseURL: "https://api.weatherbit.io/v2.0/"
});
export const key = "YOUR_API_KEY";

## serv/AppServices
import {API, key} from '@/services/api'

export default {
  getCurrentWeather (params) {
    return API.get(`current?key=${key}&city=${params.city}&country=${params.country}`)
  },
  getForecastWeather (params) {
    return API.get(`forecast/daily?key=${key}&city=${params.city}&country=${params.country}&days=${params.days}`)
  }
}
### App.vue
<template>
  <div id="app">
    <div class="ui segments">

      <Header></Header>

      <div class="ui grey segment main">
        <div class="loading" v-if="loading">Loading...</div>
        <Autocomplete 
          :items="items" 
          :value="init.city" 
          @item-selected="getData"
        ></Autocomplete>  
        <WeatherInfo 
          :loading="loading" 
          @change-scale="setScale"
        ></WeatherInfo>
        <ForecastList></ForecastList> 
      </div>

    </div>

    <Footer></Footer>   
  </div>
</template>

<script>
import Header from '@/components/Header'
import Autocomplete from '@/components/Autocomplete'
import WeatherInfo from '@/components/WeatherInfo'
import ForecastList from '@/components/ForecastList'
import Footer from '@/components/Footer'
import { initCity, cities } from '@/config'

export default {
  name: 'app',
  components:{
    Header,
    Autocomplete,
    WeatherInfo,
    ForecastList,
    Footer    
  },
  data () {
    return {
      init: initCity,
      items: cities,
      loading: true
    }
  },
  mounted: function(){
    this.getData(this.init);
  },
  methods: {
    async getData(valueSelected){
      this.loading = true
      await this.$store.dispatch('FETCH_WEATHER', valueSelected)
      this.loading = false
    },
    setScale(scale){
      this.$store.dispatch('SET_SCALE', scale)
    }
  }
}
</script>

<style>
body{
  margin: 0;
}
#app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  position: relative;
  margin: 1em auto;  
  max-width: 500px;    
  color: #fff;
}
.header{
  background-color: #131517 !important;
}
.logo{
  width: 25px;
  height: 25px;
}
.main{
  background-color: #1b1f21 !important;  
  min-height: 190px;
}
.footer {
    margin-top: 50px;
    text-align: center;
}
.loading{
  z-index: 999;
  top: 0;
  right: 0;
  position: absolute;
  padding: 5px;
  color: #fff;
  background-color: #0a0;
}
</style>
## weather
export const weatherMixins = {
  filters: {
    tempScale(value, scale) {
      return (scale == 'C') ? value : Number(((value * 9/5) + 32).toFixed(1))
    }
  }
}
