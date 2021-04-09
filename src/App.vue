<template>
  <div class="wrapper">
    <header class="header">
      <h1 class="title">Weather Forecast</h1>
      <div class="search">
        <input
          v-model="weatherTarget"
          @keydown.enter="search"
          @input="error = false"
          type="text"
          class="search__input"
          placeholder="Search city..."
        >
        <button
          class="search__button"
          @click.prevent="search"
        >
        </button>
      </div>
      <div
        v-show="error"
        class="error"
      >
        Error! <span class="error__target">
        {{ weatherTarget }}</span> not found!
      </div>
    </header>
    <main class="main">
      <div class="weather">
        <div
          v-for="(field, idx) in fields"
          :key="idx"
          :class="background"
          class="weather__item"
        >
          <span class="weather__value" v-html="field.value"></span>
          <h2 class="weather__title">{{ field.title }}</h2>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: 'WeatherApp',
  components: {
  },
  created() {
    document.title = 'Weather Forecast APP';
  },
  data() {
    return {
      fields: [
        { title: 'City', value: '-' },
        { title: 'Country', value: '-' },
        { title: 'Time', value: '-' },
        { title: 'Temperature', value: '-' },
        { title: 'Feels Like', value: '-' },
        { title: 'Weather Description', value: '-' },
        { title: 'Wind', value: '-' },
        { title: 'Clouds', value: '-' },
        { title: 'Humidity', value: '-' },
        { title: 'Pressure', value: '-' },
        { title: 'Visibility', value: '-' },
      ],
      weatherTarget: '',
      background: 'bg1',
      error: false,
    };
  },
  methods: {
    async search() {
      if (this.weatherTarget.replace(/[\s,\d]/g, '').length < 3) {
        this.setError();
        return;
      }
      const weather = await this.searchWeather(this.weatherTarget).catch();
      if (weather.cod !== 200) {
        this.setError();
        return;
      }
      this.setFields(weather);
      this.changeBackground();
      this.weatherTarget = '';
    },
    changeBackground() {
      let bg = this.getRandomImage();
      while (bg === this.background) {
        bg = this.getRandomImage();
      }
      this.background = bg;
    },
    getRandomImage() {
      return `bg${Math.floor(Math.random() * 10) + 1}`;
    },
    async searchWeather(target) {
      const API_KEY = '877daef8b2aa82cb76a4f991ed1a3dee';
      const f = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${target}&units=metric&APPID=${API_KEY}`,
      );
      const responce = await f.json();
      return responce;
    },
    setFields(weather) {
      const [city, country, time, temperature, feelsLike, weatherDescription,
        wind, clouds, humidity, pressure, visibility] = this.fields;
      const {
        name, sys: { country: APIcountry }, main: {
          temp: APItemperature, feels_like: APIfeelsLike, pressure: APIpressure,
          humidity: APIhumidity,
        }, weather: [APIweather], clouds: { all: APIclouds }, visibility: APIvisibility,
        wind: { speed: APIwindSpeed, deg: APIwindDeg },
      } = weather;
      city.value = name;
      country.value = APIcountry;
      temperature.value = `${Math.round(APItemperature * 10) / 10}°C`;
      feelsLike.value = `${Math.round(APIfeelsLike * 10) / 10}°C`;
      humidity.value = `${APIhumidity} %`;
      pressure.value = `${Math.round(APIpressure / 1.33)} mm Hg`;
      weatherDescription.value = APIweather.description;
      clouds.value = `${APIclouds} %`;
      visibility.value = `${(APIvisibility / 1000).toFixed(1)} km`;
      wind.value = `${APIwindSpeed} m/s ${this.windDirection(APIwindDeg)}`;
      time.value = this.getRequestDate();
    },
    windDirection(deg) {
      if (deg > 22.5 && deg <= 67.5) return '&#129151; NE';
      if (deg > 67.5 && deg <= 112.5) return '&#129144; E';
      if (deg > 112.5 && deg <= 157.5) return '&#129148; SE';
      if (deg > 157.5 && deg <= 202.5) return '&#129145; S';
      if (deg > 202.5 && deg <= 236.5) return '&#129149; SW';
      if (deg > 236.5 && deg <= 292.5) return '&#129146; W';
      if (deg > 292.5 && deg <= 337.5) return '&#129150; NW';
      return '&#129147; N';
    },
    getRequestDate() {
      const hm = new Date()
        .toLocaleString('en-US', {
          hour: 'numeric',
          hour12: true,
          minute: 'numeric',
        })
        .toLowerCase()
        .replace(/\s/g, '');
      const dm = new Date()
        .toLocaleString('en-US', {
          day: 'numeric',
          month: 'long',
        });
      return `${hm}, ${dm}`;
    },
    setError() {
      this.error = true;
      this.fields.forEach((field) => {
        field.value = '-';
      });
    },
  },
};
</script>

<style lang="sass">
@import '@/assets/sass/reset.sass'
@import '@/assets/sass/mixins.sass'
@import '@/assets/sass/style.sass'

+font-face(Itim, itim)

#app
  font-family: Itim, cursive
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  background-color: #191919
  color: #fff
  height: 100vh
  overflow: hidden
  width: 100%

.header
  height: 96px
  display: flex
  justify-content: flex-start
  align-items: center
  padding: 0 8vw

.title
  font-size: 1.4em
  line-height: 1.4em
  font-weight: 400
  color: #fff
  padding-right: 2vh
  +r(560)
    padding-right: 0
  +r(400)
    font-size: 1.2em

.error
  font-size: 1em
  padding-left: 2vh
  +r(560)
    font-size: 0.8em

.error__target
  color: red

.search
  display: flex
  justify-content: flex-start
  align-items: center

.search__input
  max-width: 320px
  width: 100%
  height: 40px
  padding: 10px 20px
  font-size: 1.25em
  font-family: Itim, cursive
  color: #191919
  cursor: text
  border-radius: 0
  +placeholder
    color: #191919
  +r(400)
    font-size: 1em

.search__button
  width: 40px
  height: 40px
  text-align: center
  line-height: 40px
  background-color: #eee
  border-radius: 0
  background: #eee url(~@/assets/img/magnifying-glass.png) no-repeat 10px 12px
  transition: background-color 300ms ease-in-out
  &:hover
    background-color: #888

.main
  height: calc(100vh - 96px)

.weather
  display: grid
  grid-template-columns: repeat(4, 1fr)
  grid-gap: 10px

.weather > .weather__item
  height: calc((100vh - 96px - 30px) / 4)
  background-size: cover
  background-attachment: fixed
  display: flex
  flex-direction: column
  align-items: center
  justify-content: center
  text-align: center
  transition: all 700ms ease-in-out
  cursor: default

.weather__item
  &:nth-of-type(3)
    grid-column: 3 / 5
  &:nth-of-type(6)
    grid-column: 3 / 5
  &:nth-of-type(7)
    grid-column: 1 / 3
  &:nth-of-type(10)
    grid-column: 1 / 3
  &:nth-of-type(11)
    grid-column: 3 / 5

.bg1
  background: url(~@/assets/img/bg1.webp)
.bg2
  background: url(~@/assets/img/bg2.webp)
.bg3
  background: url(~@/assets/img/bg3.webp)
.bg4
  background: url(~@/assets/img/bg4.webp)
.bg5
  background: url(~@/assets/img/bg5.webp)
.bg6
  background: url(~@/assets/img/bg6.webp)
.bg7
  background: url(~@/assets/img/bg7.webp)
.bg8
  background: url(~@/assets/img/bg8.webp)
.bg9
  background: url(~@/assets/img/bg9.webp)
.bg10
  background: url(~@/assets/img/bg10.webp)

.weather__value
  font-size: 2.4em
  text-shadow: 2px 2px #191919
  margin-bottom: 30px
  +r(768)
    font-size: 1.6em
    margin-bottom: 20px
  +r(480)
    font-size: 1.2em
  +r(400)
    font-size: 1em

.weather__title
  font-size: 1.4em
  text-shadow: 2px 2px #191919
  +r(768)
    font-size: 1.2em
  +r(480)
    font-size: 1em
  +r(400)
    font-size: 0.8em

</style>
