<template>
  <div id="app">
    <h1>Predict-a-bike</h1>
    <p>Availability prediction for <a href="https://kaupunkipyorat.hsl.fi/en">Helsinki city bikes.</a> Try July to October.</p>
    <PredictionControl/>
  </div>
</template>

<script>
import PredictionControl from './components/PredictionControl.vue'

export default {
  name: 'app',
  components: {
    PredictionControl
  }
}
</script>

<style>
#app {
  font-family: 'Gotham Rounded Medium';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #3d454f;
}

html {
  height: 100%;
  background-image: linear-gradient(#ffffff23, #fcbc1917);
}

h1 {
  font-size: 2em;
  font-weight: normal;
  font-family: 'Gotham Rounded Bold';
}

h2 {
  font-size: 1.2em;
}

p {
  margin-left: 1em;
  margin-right: 1em;
}

@font-face {
font-family: 'Gotham Rounded Medium';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Medium'), url('./assets/fonts/GothamRounded-Medium.woff') format('woff');
}


@font-face {
font-family: 'Gotham Rounded Medium Italic';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Medium Italic'), url('./assets/fonts/GothamRounded-MediumItalic.woff') format('woff');
}


@font-face {
font-family: 'Gotham Rounded Bold';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Bold'), url('./assets/fonts/GothamRounded-Bold.woff') format('woff');
}


@font-face {
font-family: 'Gotham Rounded Bold Italic';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Bold Italic'), url('./assets/fonts/GothamRounded-BoldItalic.woff') format('woff');
}


@font-face {
font-family: 'Gotham Rounded Light';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Light'), url('./assets/fonts/GothamRounded-Light.woff') format('woff');
}


@font-face {
font-family: 'Gotham Rounded Light Italic';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Light Italic'), url('./assets/fonts/GothamRounded-LightItalic.woff') format('woff');
}


@font-face {
font-family: 'Gotham Rounded Book';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Book'), url('./assets/fonts/GothamRounded-Book.woff') format('woff');
}


@font-face {
font-family: 'Gotham Rounded Book Italic';
font-style: normal;
font-weight: normal;
src: local('Gotham Rounded Book Italic'), url('./assets/fonts/GothamRounded-BookItalic.woff') format('woff');
}
</style>
