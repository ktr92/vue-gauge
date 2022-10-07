<template>
  <div id="app">
   
    <div class="content">
    

      <div class="lets-play">
     
        <VueSvgGauge
          class="customizable-gauge"
          :start-angle="parseInt(startAngle || 130)"
          :end-angle="parseInt(endAngle || -130)"
          :value="parseInt(value)"
          :separator-step="parseInt(separatorStep)"
          :min="parseInt(min || 0)"
          :max="parseInt(max || 100)"
          :scale-interval="parseInt(scale)"
          :inner-radius="parseInt(innerRadius)"
          :separator-thickness="parseInt(separatorThickness)"
          :base-color="baseColor"
          :gauge-color="[ { offset: 0, color: '#FD2232' },
          { offset: 150, color: '#FCEC68' },
          { offset: 594, color: '#6FE566' },
          { offset: 904, color: '#5BB657' },
          { offset: 999, color: '#fff' }]"
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
        startAngle: -130,
        endAngle: 130,
        value: 620,
        separatorStep: 0,
        min: 0,
        max: 999,
        scale: 5,
        innerRadius: 65,
        separatorThickness: 4,
        baseColor: '#d0cdcd',
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
      /**
       * Text to display according to the random value
       * @type {String}
       */
      text() {
        const { random } = this

        if (random < 25) {
          return 'Very Bad 🙁'
        }
        if (random < 50) {
          return 'Pretty ok 😶'
        }
        if (random < 75) {
          return 'Gets nice 🙂'
        }
        return 'Super good 😄'
      }
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

<style lang="stylus">
  body {
    background-color: #f8f8f8
  }

  #app {
    font-family: 'Roboto Mono', Monaco, courier, monospace
    max-width: 980px
    margin: auto
    text-align: center
    padding-top: 40px

    h3 { margin-bottom: 30px }

    a.button {
      display: inline-block
      width: 240px
      margin: 0.5em
      font-family: 'Roboto Mono', Monaco, courier, monospace
      font-weight: 700
      color: #fff
      background-color: #83d1a2
      border-bottom: 2px solid #64bf8a
      padding: 12px 14px
      border-radius: 4px
      transition: all 0.15s ease
      text-decoration: none

      &:hover {
        background-color: #8CDFAD
        border-bottom: 2px solid #6FCF97
      }
    }

    .content {
      background-color: white
      border: 1px solid #eee
      margin: 30px 0
      padding: 20px
    }

    .mutliple-examples {
      background-color: white
      display: flex
      justify-content: center
      align-items: center
      border-radius: 10px
      margin: 20px auto

      > *:not(:last-child) {
        margin-right: 30px
      }
    }

    .lets-play {
      display: flex
      flex-direction: column
      justify-content: center
      align-items: center

      h2 {
        font-size: 27px
        margin-top: 0
      }

      .customizable-gauge {
        max-width: 300px
      }

      .customizer-title {
        font-size: 20px
        margin: 10px 0 0 0
      }

      .customizer {
        margin-top: 40px
        display: flex
        justify-content: center
        flex-wrap: wrap

        > *:not(:last-child) {
          margin-right: 20px
        }
      }
    }

    .mini-gauge {
      max-width: 180px

      .inner-text {
        width: 100%
        height: 100%
      }
    }

    .inner-text {
      &--1, &--3 {
        display: flex
        justify-content: center
        margin-top: 85px
        font-size: 20px
        color: #de3a21
        font-weight: bold

        span { max-width: 100px }
      }

      &--3 {
        margin-top: 70px
      }

      &--2 {
        display: flex
        align-items: center
        justify-content: center
        font-size: 20px
        color: #de3a21
        font-weight: bold

        span { max-width: 90px }
      }
    }

    .block {
      display: flex

      .property:first-child { margin-right: 20px }
    }

    .property {
      display: flex
      flex-direction: column
      align-items: flex-start
      margin-bottom: 15px

      label { margin-bottom: 3px }
    }

    .bottom {
      margin: 30px 0

      a {
        color: black
        font-weight: bold
        text-decoration: none
      }
    }

    input[type=text], input[type=number], .select-container select {
      width: 100%
      padding: 5px 20px
      box-sizing: border-box
      font-size: 16px
      max-width: 150px
    }

    .select-container {
    	position: relative
    	background-color: #fff
    	border: #d4d4d4 1px solid
    	overflow: hidden
      width: 150px
    }
    .select-container select {
      background-color: #fff
    	-webkit-appearance: none
    	-moz-appearance: none
    	appearance: none
    	width: 110%
    	height: auto
    	border: 0
    	margin: 0
    	border-radius: 0
    	overflow: hidden
    	text-overflow: ellipsis
    }
    .select-container::after {
    	content: ''
    	position: absolute
    	top: 50%
    	margin-top: -3px
    	right: .75em
    	display: block
    	width: 0
      height: 0
    	border-color: transparent
    	border-top-color: #444
    	border-width: 6px
    	border-style: solid
    	pointer-events: none
    }
  }
</style>
