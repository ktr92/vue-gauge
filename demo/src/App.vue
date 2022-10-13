<template>
  <div id="app">
   
    <div class="content">
    

      <div class="lets-play">
     
        <VueSvgGauge
          class="customizable-gauge tachometer-main"
          :start-angle="parseInt(startAngle || 130)"
          :end-angle="parseInt(endAngle || -130)"
          :value="parseInt(mainGauge.value)"
          :main-radius="parseInt(mainGauge.mainRadius)"
          :center="parseInt(mainGauge.mainRadius)"
          :inner-radius="parseInt(mainGauge.innerRadius)"
          :separator-step="parseInt(separatorStep)"
          :min="parseInt(min || 0)"
          :max="parseInt(max || 100)"
          :scale-interval="parseInt(scale)"
          :separator-thickness="parseInt(separatorThickness)"
          :base-color="baseColor"
          :diff="0"
          :is-value="true"
          :gauge-color="colors"
          :easing="easing"
        />

        
        <VueSvgGauge
          class="customizable-gauge tachometer-scale"
          :start-angle="parseInt(startAngle || 130)"
          :end-angle="parseInt(endAngle || -130)"
          :value="parseInt(mainGauge.outerGauge)"
          :separator-step="parseInt(separatorStep)"
          :min="parseInt(min || 0)"
          :max="parseInt(max || 100)"
          :scale-interval="parseInt(scale)"
          :center="parseInt(outerGauge.mainRadius -1)"
          :inner-radius="parseInt(outerGauge.innerRadius)"
          :main-radius="parseInt(outerGauge.mainRadius)"
          :diff="parseInt(outerGauge.diff)"
          :outer="parseInt(outerGauge.outer)"
          :is-value="false"
          :separator-thickness="parseInt(separatorThickness)"
          :base-color="baseColor"
          :gauge-color="colors"
          :easing="easing"
        />


      
      
      </div>
    </div>

  </div>
</template>

<script>
  export default {
    name: 'app',
    data() {
      return {
        colors: [
          { offset: 0, color: '#FD2232' },
          { offset: 150, color: '#FCEC68' },
          { offset: 594, color: '#6FE566' },
          { offset: 904, color: '#5BB657' },
          { offset: 999, color: '#fff' }
        ],
        mainGauge: {
          value:700,
          innerRadius: 75,
          mainRadius: 114,
        },
        outerGauge: {
          value:999,
          innerRadius: 118,
          mainRadius: 122,
          diff: 4,
          outer: 4
        },
        startAngle: -130,
        endAngle: 130,
        separatorStep: 0,
        min: 0,
        max: 999,
        scale: 5,
        separatorThickness: 4,
        baseColor: '#fff',
        easingFct: 'Circular',
        easingType: 'Out',
        random: 30,
      }
    },
    computed: {
      easing() {
        const { easingFct, easingType } = this

        return [easingFct, easingType].join('.')
      },
      
    },
    /**
     * Change the random value for the example gauges
     * @returns {void}
     */
    mounted() {
      setInterval(() => {
        const min = 0
        const max = 100
        this.random = Math.random() * (+max - +min) + +min;
      }, 4000)
    }
  }
</script>

<style>
 body {
  background-color: #f8f8f8;
}
#app {
  font-family: 'Roboto Mono', Monaco, courier, monospace;
  max-width: 980px;
  margin: auto;
  text-align: center;
  padding-top: 40px;
}
#app h3 {
  margin-bottom: 30px;
}
#app a.button {
  display: inline-block;
  width: 240px;
  margin: 0.5em;
  font-family: 'Roboto Mono', Monaco, courier, monospace;
  font-weight: 700;
  color: #fff;
  background-color: #83d1a2;
  border-bottom: 2px solid #64bf8a;
  padding: 12px 14px;
  border-radius: 4px;
  transition: all 0.15s ease;
  text-decoration: none;
}
#app a.button:hover {
  background-color: #8cdfad;
  border-bottom: 2px solid #6fcf97;
}
#app .content {
  background-color: #fff;
  border: 1px solid #eee;
  margin: 30px 0;
  padding: 20px;
}
#app .mutliple-examples {
  background-color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 10px;
  margin: 20px auto;
}
#app .mutliple-examples > *:not(:last-child) {
  margin-right: 30px;
}
#app .lets-play {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
#app .lets-play h2 {
  font-size: 27px;
  margin-top: 0;
}
#app .lets-play .customizer-title {
  font-size: 20px;
  margin: 10px 0 0 0;
}
#app .lets-play .customizer {
  margin-top: 40px;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
}
#app .lets-play .customizer > *:not(:last-child) {
  margin-right: 20px;
}
#app .mini-gauge {
  max-width: 180px;
}
#app .mini-gauge .inner-text {
  width: 100%;
  height: 100%;
}
#app .inner-text--1,
#app .inner-text--3 {
  display: flex;
  justify-content: center;
  margin-top: 85px;
  font-size: 20px;
  color: #de3a21;
  font-weight: bold;
}
#app .inner-text--1 span,
#app .inner-text--3 span {
  max-width: 100px;
}
#app .inner-text--3 {
  margin-top: 70px;
}
#app .inner-text--2 {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  color: #de3a21;
  font-weight: bold;
}
#app .inner-text--2 span {
  max-width: 90px;
}
#app .block {
  display: flex;
}
#app .block .property:first-child {
  margin-right: 20px;
}
#app .property {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin-bottom: 15px;
}
#app .property label {
  margin-bottom: 3px;
}
#app .bottom {
  margin: 30px 0;
}
#app .bottom a {
  color: #000;
  font-weight: bold;
  text-decoration: none;
}
#app input[type=text],
#app input[type=number],
#app .select-container select {
  width: 100%;
  padding: 5px 20px;
  box-sizing: border-box;
  font-size: 16px;
  max-width: 150px;
}
#app .select-container {
  position: relative;
  background-color: #fff;
  border: #d4d4d4 1px solid;
  overflow: hidden;
  width: 150px;
}
#app .select-container select {
  background-color: #fff;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  width: 110%;
  height: auto;
  border: 0;
  margin: 0;
  border-radius: 0;
  overflow: hidden;
  text-overflow: ellipsis;
}
#app .select-container::after {
  content: '';
  position: absolute;
  top: 50%;
  margin-top: -3px;
  right: 0.75em;
  display: block;
  width: 0;
  height: 0;
  border-color: transparent;
  border-top-color: #444;
  border-width: 6px;
  border-style: solid;
  pointer-events: none;
}
/*  .customizable-gauge {
        max-width: 240px
      }
 */
#app .lets-play .tachometer-scale {
  position: absolute;
  top: 50%;
  left: 50%;
/*   transform: translate(-137px, -121px);
 */  transform: translate(-50%, -50%);
  margin-top: -1px;
}


</style>

<style>
  .tachometer-main {
    background-image: url('/src/img/scale.svg')
  }
</style>
