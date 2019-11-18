<template>
    <div class="contaner_wrapper">
        <header class="city">
            <div class="change_city ">
                <h2>{{selectedCity.name}}</h2>
                <div class="caption on_hover" @click="citybox = !citybox">Сменить город</div>
                <ul class="change_city_list" :class="{'hide': !citybox }" >
                    <li class="search"><input type="text" v-model="findcity" autofocus></li>
                    <li v-for="item in citylist" @click="selectCity(item)" v-if="!item.name.toLowerCase().indexOf(findcity.toLowerCase())">{{item.name}} ({{item.regionName}})</li>
                </ul>
                <div class="find_place on_hover" @click="getLocation()">Мое местоположение</div>
            </div>
            <div class="temp_type on_hover" @click="changeTempType()">
                <span>º</span>
                <p class="c" :class="{'active': temptype}" >C</p>
                <p class="f" :class="{'active': !temptype}" >F</p>
            </div>
        </header>
        <main class="weather">
            <div class="temp">{{mainTemp}}º</div>
            <div class="pic">
                <img :src="getMainWeather.link" alt="weather img">
                <div class="temp_caption">{{getMainWeather.desc}}</div>
            </div>

        </main>
        <footer class="weather_info">
            <div><h3 class="info">Ветер</h3><p class="info_details">{{windSpeed}} м/c, {{windDeg}}</p></div>
            <div><h3 class="info">Давление</h3><p class="info_details">{{getPressure}} мм рт. ст.</p></div>
            <div><h3 class="info">Влажность</h3><p class="info_details">{{tempnow.main.humidity}}%</p></div>
            <div><h3 class="info">Вероятность осадков</h3><p class="info_details">{{tempnow.clouds.all}}%</p></div>
        </footer>
    </div>
</template>

<script>
    import axios from 'axios'
    import citylist from './citylist'
    const openweatherapikey = "6aa63b4074313795d4771a820bdef2a1"


    export default {
        name: "weather",
        data(){
            return{
                temptype: true,
                findcity: '',
                cords: {
                    lat: 0,
                    lng: 0,
                    accuracy: 0
                },
                tempnow:{
                    weather: [
                        {main: '',description: ''}
                    ],
                    main: {
                        temp: {
                            type: Number,
                            default: null
                        },
                        pressure: 0
                    },
                    clouds:{
                        all: 0
                    },
                    wind:{
                        speed: 0,
                        deg: 0
                    }
                },
                selectedCity: {
                    name: 'Местоположение'
                },
                citybox: false,
                citylist,
            }
        },
        methods:{
            changeTempType(){
                if(this.temptype){
                    this.temptype = false
                }else{
                    this.temptype = true
                }
            },
            getLocation(){
                this.$getLocation()
                    .then(coordinates => {
                        this.cords.lng = coordinates.lng
                        this.cords.lat = coordinates.lat
                        this.cords.accuracy = coordinates.accuracy
                    }).catch(res=>{
                    console.log('getLocation error',res)
                })
                this.$watchLocation()
                    .then(coordinates => {
                        this.cords.lng = coordinates.lng
                        this.cords.lat = coordinates.lat
                        this.cords.accuracy = coordinates.accuracy
                        this.getWeather()
                    }).catch(res=>{
                    console.log('watchLocation error',res)
                })
            },
            getWeather(){
                axios.get('http://api.openweathermap.org/data/2.5/weather?lat='+this.cords.lat+'&lon='+this.cords.lng+'&appid='+openweatherapikey+'&lang=ru')
                    .then(res=>{
                        this.tempnow = res.data
                    }).catch(res=>{
                        console.warn('getWeather error', res)
                    })
            },
            getCityWeather(){
                axios.get('http://api.openweathermap.org/data/2.5/weather?q='+this.selectedCity.name+'&appid='+openweatherapikey+'&lang=ru')
                    .then(res=>{
                        this.tempnow = res.data
                    }).catch(res=>{
                    console.warn('getWeather error', res)
                    })
            },
            selectCity(city){
                this.selectedCity = city
                this.citybox = false
                this.getCityWeather()
            },

        },
        computed:{
            mainTemp(){
                  if(this.temptype){
                      if(this.tempnow.main.temp!=null)
                      return (this.tempnow.main.temp - 273).toFixed(0)
                  }else{
                      if(this.tempnow.main.temp!=null)
                      return ((this.tempnow.main.temp * 1.8) - 459).toFixed(0)
                  }
              },
            windSpeed(){
              return (this.tempnow.wind.speed).toFixed(0)
            },
            windDeg(){
                let deg = this.tempnow.wind.deg
                if(deg>340&&deg<20){
                    return 'Северный'
                }else if(deg>20&&deg<60){
                    return 'Северо-восточный'
                }else if(deg>60&&deg<110){
                    return 'Восточный'
                }else if(deg>110&&deg<150){
                    return 'Юго-Восточный'
                }else if(deg>150&&deg<200){
                    return 'Южный'
                }else if(deg>200&&deg<250){
                    return 'Юго-Западный'
                }else if(deg>250&&deg<290){
                    return 'Западный'
                }else if(deg>290&&deg<340){
                    return 'Северо-Западный'
                }

            },
            getPressure(){
                return (this.tempnow.main.pressure * 7.501/10).toFixed(0) //hPa в мм/рс нашел на форуме коэффициент
            },
            getMainWeather(){
                let description = this.tempnow.weather[0].description
                let main = this.tempnow.weather[0].main
                let link = ''
                if(main == 'Clear'){
                    link = '/static/img/sun.png'
                }else if(main=='Rain'){
                    link = '/static/img/rain.png'
                }else if(main=='Snow'){
                    link = '/static/img/snow.png'
                }else if(main=='Extreme'){
                    link = '/static/img/storme.png'
                }else if(main=='Clouds'){
                    link = '/static/img/cloud.png'
                }else if(main=='Mist'){
                    link = '/static/img/mist.png'
                }
                console.log('link', link, 'desc', description)
                return {link: link, desc: description}
            },


        },
        mounted(){
            this.getLocation()
        }

    }
</script>

<style scoped>

</style>