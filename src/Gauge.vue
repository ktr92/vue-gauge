<template>
  <div class="gauge" :style="[
    maxWidth,
    marginTop,
    outer ? {left: `calc(50% + ${diff/outer}px)`} : {}
  ]">
    <svg
      v-if="height"
      :viewBox="`0 0 ${RADIUS * 2} ${RADIUS * 2}`" 
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <!-- This puts an inner shadow on the empty part of gauge -->
        <filter :id="`innershadow-${_uid}`">
          <!-- <feFlood flood-color="#ECEEF6" />
          <feComposite in2="SourceAlpha" operator="out" />
          <feGaussianBlur stdDeviation="0" result="blur" />
          <feComposite operator="atop" in2="SourceGraphic" /> -->

           <feOffset
              dx='0'
              dy='0'
            />

            <!-- Shadow blur -->
            <feGaussianBlur
              stdDeviation='26  '
              result='offset-blur'
            />

            <!-- Invert drop shadow to make an inset shadow -->
            <feComposite
              operator='out'
              in='SourceGraphic'
              in2='offset-blur'
              result='inverse'
            />
            
            <!-- Cut color inside shadow -->
            <feFlood
              flood-color='#ECEEF6'
              flood-opacity='1'
              result='color'
            />
            <feComposite
              operator='in'
              in='color'
              in2='inverse'
              result='shadow'
            />

            <!-- Placing shadow over element -->
            <feComposite
              operator='over'
              in='shadow'
              in2='SourceGraphic'
              result='shadow'
            />
        </filter>



        <!-- Determine the gradient color on the full part of the gauge -->
        <template v-if="hasGradient">
          <linearGradient 
          v-if="hasGradient"
          :id="`gaugeGradient-${_uid}`"
        >
          <stop
            v-for="(color, index) in gaugeColor"
            :key="`${color.color}-${index}`"
            :offset="`${color.offset}%`" :stop-color="color.color"
          />
        </linearGradient>
        </template>
        

        <mask :id="`innerCircle-${_uid}`">
          <!-- Mask to make sure only the part inside the circle is visible -->
          <!-- RADIUS - 0.5 to avoid any weird display -->
          <circle :r="RADIUS - 0.5" :cx="this.X_CENTER" :cy="this.Y_CENTER" fill="white" />

          <!-- Mask to remove the inside of the gauge -->
          <circle :r="innerRadius" :cx="this.X_CENTER" :cy="this.Y_CENTER" fill="black" />

          <template v-if="separatorPaths">
            <!-- Mask for each separator -->
            <path
              v-for="(separator, index) in separatorPaths"
              :key="index"
              :d="separator" fill="black"
            />
          </template>
        </mask>
      </defs>

     <g :mask="`url(#innerCircle-${_uid})`"> 
        <!-- Draw a circle if the full gauge has a 360° angle, otherwise draw a path -->
        <circle
          v-if="isCircle"
          :r="RADIUS" :cx="this.X_CENTER" :cy="this.Y_CENTER"
          :fill="hasGradient ? `url(#gaugeGradient-${_uid})` : gaugeColor"
        />
        <path
          v-for="(color, index) in gaugeColor"
          :key="`${color.color}-${index}`"
          :d="basePathM(
            getAngle(color.offset), 
            getAngle(index >= gaugeColor.length - 1 ? max : gaugeColor[index + 1].offset))" 
            :fill="color.color"
          >
        </path>
       

        <!-- Draw a circle if the empty gauge has a 360° angle, otherwise draw a path -->
        <circle
          v-if="value === min && isCircle"
          :r="RADIUS" :cx="X_CENTER" :cy="Y_CENTER"
          :fill="baseColor"
        />
        <template v-else>
           <path v-if="isValue === true" :d="gaugePathCover" :fill="baseColor" :filter="`url(#innershadow-${_uid})`" />
        </template>
       
      </g>

      <template v-if="scaleLines">
        <!-- Display a line for each tick of the scale -->
       <!--  <line
          v-for="(line, index) in scaleLines"
          :key="`${line.xE}-${index}`"
          :x1="line.xS" :y1="line.yS" :x2="line.xE" :y2="line.yE"
          stroke-width="3" stroke="#fff"
        /> -->
      </template>

      <!-- This allow to display html inside the svg -->
      <foreignObject x="0" y="0" width="100%" :height="height">
        <slot />
      </foreignObject>
    </svg>

    <div id="taho-overlay" v-if="isValue">
      <div 
        v-for="(step, index) in stepsCoords" 
        :key="step.x" 
        class="taho-overlay-onchart-digit" 
        :style="{'left': step.x + 'px', 'top': step.y + 'px'}"
      >{{ gaugeColor[index].offset }}</div>
      
      <div id="taho-overlay-main-digit">{{ value }}</div>
      <div id="taho-overlay-description"> 
        <span>{{ desctext }}</span>
        <i class="icon pi-exclamation-circle pi"></i> 
      </div>
  </div>
  </div>
</template>

<script>
  import TWEEN from '@tweenjs/tween.js'
  import _get from 'lodash/get'

  // Main radius of the gauge

  // Coordinates of the center based on the radius
/*   const X_CENTER = 100
  const Y_CENTER = 100 */

  /**
   * Turn polar coordinate to cartesians
   * @param   {Number} centerX - abscisse of the center
   * @param   {Number} centerY - ordinate of the center
   * @param   {Number} radius  - radius of the circle
   * @param   {Number} angle   - angle in degres
   * @param   {Number} xcenter   
   * @param   {Number} ycenter   
   * @returns {String}         - d property of the path
   */
  function polarToCartesian(xcenter, ycenter, radius, angle) {
    const angleInRadians = (angle - 90) * Math.PI / 180

    return {
      x: xcenter + (radius * Math.cos(angleInRadians)),
      y: ycenter + (radius * Math.sin(angleInRadians)),
    }
  }
  function polarToCartesianPoint(xcenter, ycenter, radius, angle) {
    const angleInRadians = (angle - 90) * Math.PI / 180

    const x =  xcenter + (radius * Math.cos(angleInRadians))
    const y = ycenter + (radius * Math.sin(angleInRadians))

    if (x > 90) {
      if (y > 90) {
        // right bottom
        return {
          x: x + 10,
          y: y
        }  
      }
      else {
        // right top
        return {
          x: x,
          y: y - 20
        }  
      }
    }
    else {
       if (y > 130) {
        // left bottom
        return {
          x: x - 25,
          y: y + 5
        }  
      }
      else {
        // left top
        return {
          x: x - 35,
          y: y - 5
        }  
      }
    }
  }

  /**
   * Describe a gauge path according
   * @param   {Number} radius
   * @param   {Number} startAngle - in degre
   * @param   {Number} endAngle   - in degre
   * @returns {String}            - d property of the path
   */
  
  function describePath(xcenter, ycenter, radius, startAngle, endAngle) {
    const start = polarToCartesian(xcenter, xcenter, radius, endAngle)
    const end = polarToCartesian(xcenter, xcenter, radius, startAngle)

    const largeArcFlag = endAngle - startAngle <= 180 ? '0' : '1'

    const d = [
      'M', start.x, start.y,
      'A', radius, radius, 0, largeArcFlag, 0, end.x, end.y,
      'L', xcenter, ycenter,
    ].join(' ')

    return d
  }

  function describePathCover(mainR, innerR, xcenter, ycenter, radius, startAngle, endAngle) {
    const start = polarToCartesian(xcenter, xcenter, radius, endAngle)
    const end = polarToCartesian(xcenter, xcenter, radius, startAngle)

    const largeArcFlag = endAngle - startAngle <= 180 ? '0' : '1'

    const angle = endAngle - startAngle

    const w = mainR - innerR

   /*  const d = [
      'M', start.x, start.y,
      'A', radius, radius, 0, largeArcFlag, 0, end.x, end.y,
      'L', polarToCartesian(xcenter, xcenter, radius - w , startAngle).x,
           polarToCartesian(xcenter, xcenter, radius - w, startAngle).y,
      'A', radius, radius, 0, largeArcFlag, 1, polarToCartesian(xcenter, xcenter, innerR, endAngle).x,
           polarToCartesian(xcenter, xcenter, innerR, endAngle).y,

    ].join(' ') */

     const d = [
      'M', start.x, start.y,
      'A', radius, radius, 0, largeArcFlag, 0, end.x, end.y,
      'L', xcenter, ycenter,
    ].join(' ')

    return d
  }

  function getCoordFromDegrees(angle, radius, svgSize) {
    const x = Math.cos(angle * Math.PI / 180);
    const y = Math.sin(angle * Math.PI / 180);
    const coordX = x * radius + svgSize / 2;
    const coordY = y * -radius + svgSize / 2;
    return [coordX, coordY].join(' ');
  }

  export default {
    name: 'Gauge',
    props: {
      /**
       * Gauge value
       */
      value: {
        type: Number,
        default: 70,
      },
      /**
       * Gauge min value
       */
      min: {
        type: Number,
        default: 0,
      },
      /**
       * Gauge max value
       */
      max: {
        type: Number,
        default: 100,
      },
      isValue: {
        type: Boolean,
        default: false
      },
      /**
       * Must be between -360 and 360
       * startAngle MUST be inferior to endAngle
       */
      startAngle: {
        type: Number,
        default: -90,
        validator: (value) => {
          if (value < -360 || value > 360) {
            console.warn('GaugeChart - props "startAngle" must be between -360 and 360')
          }
          return true
        },
      },
      /**
       * Must be between -360 and 360
       * startAngle MUST be inferior to endAngle
       */
      endAngle: {
        type: Number,
        default: 90,
        validator: (value) => {
          if (value < -360 || value > 360) {
            console.warn('GaugeChart - props "endAngle" must be between -360 and 360')
          }
          return true
        },
      },
      /**
       * Size of the inner radius between 0 and RADIUS
       * The closer to RADIUS, the thinner the gauge will be
       */
      innerRadius: {
        type: Number,
        default: 60
      },
      diff: {
      type: Number,
        default: 0
      },
      outer: {
        type: Number,
        default: 0
      },
      center: {
        type: Number,
        default: 60
      },
      /**
       * Separator step, will display a separator each min + (n * separatorStep)
       * Won't display any separator if 0 or null
       */
      separatorStep: {
        type: Number,
        default: 10,
        validator: (value) => {
          if (value !== null && value < 0) {
            console.warn('GaugeChart - props "separatorStep" must be null or >= 0')
          }
          return true
        },
      },
      /**
       * Separator Thickness, unit is in degree
       */
      separatorThickness: {
        type: Number,
        default: 4,
      },
      /**
       * Gauge color. Can be :
       * - a simple color if passed as a 'string'
       * - a gradient if is an array of objects :
       * { offset: percentage where the color starts, color: color to display }
       */
      gaugeColor: {
        type: [Array, String],
        default: () => ([
          { offset: 0, color: '#FD2232' },
          { offset: 150, color: '#FCEC68' },
          { offset: 594, color: '#6FE566' },
          { offset: 904, color: '#5BB657' },
          { offset: 999, color: '#fff' },
        ]),
      },
      /**
       * Color of the base of the gauge
       */
      baseColor: {
        type: String,
        default: '#fff',
      },
      mainRadius: {
        type: Number,
        default: 100
      },
      /**
       * Animation easing option
       * You can check the Tween.js doc here :
       * https://github.com/tweenjs/tween.js/blob/master/docs/user_guide.md
       *
       * There are a few existing function gourped by equation they represent:
       * Linear, Quadratic, Cubic, Quartic, Quintic, Sinusoidal, Exponential,
       * Circular, Elastic, Back and Bounce
       *
       * And then by the easing type: In, Out and InOut.
       * The syntaxe is : equation.easingType
       */
      easing: {
        type: String,
        default: 'Circular.Out',
      },
      /**
       * Scale interval
       * Won't display any scall if 0 or `null`
       */
      scaleInterval: {
        type: Number,
        default: 5,
        validator: (value) => {
          if (value !== null && value < 0) {
            console.warn('GaugeChart - props "scaleInterval" must be null or >= 0')
          }
          return true
        },
      },
      /**
       * Transition duration in ms
       */
      transitionDuration: {
        type: Number,
        default: 1500,
      },
    },
    data() {
      return {
       /*  X_CENTER: this.mainRadius,
        Y_CENTER: this.mainRadius, */
        
        /**
         * Tweened value for the animation of the gauge
         * Starts at `min`
         * @type {Number}
         */
        tweenedValue: this.min,
       
      }
    },
    computed: {
      desctext () {
        if (this.value <= 179) {
           return 'минимум';
        } else if (this.value <= 623) {
            return 'низкий';
        } else if (this.value <= 912) {
            return 'средний';
        } else {
            return 'высокий';
        } 
      },
      maxWidth () {
      return {maxWidth: `${this.RADIUS * 2 - this.diff/2 + 'px'}`}
      },
      marginTop() {
        return {marginTop: `-${ this.diff + this.outer/2 }px`}
      },
      RADIUS() {
        return this.mainRadius
      },
      X_CENTER () {
      return this.center
      },
      Y_CENTER () {
      return this.center
      },
      steps () {
        return this.gaugeColor.map(item => item.offset)
      },
      stepsCoords () {
        return this.gaugeColor.map((item, index) => this.basePoint(
            this.getAngle(item.offset), 
            this.getAngle(index >= this.gaugeColor.length - 1 ? this.max : this.gaugeColor[index + 1].offset)))
      },
      /**
       * Height of the viewbox calculated by getting
       * - the lower y between the center and the start and end angle
       * - (RADIUS * 2) if one of the angle is bigger than 180°
       * @type {Number}
       */
      height() {
        const { endAngle, startAngle } = this
        const { y: yStart } = polarToCartesian(this.X_CENTER, this.Y_CENTER, this.RADIUS, startAngle)
        const { y: yEnd } = polarToCartesian(this.X_CENTER, this.Y_CENTER, this.RADIUS, endAngle)

        return Math.abs(endAngle) <= 180 && Math.abs(startAngle) <= 180
          ? Math.max(this.Y_CENTER, yStart, yEnd)
          : this.RADIUS * 2
      },
      /**
       * d property of the path of the base gauge (the colored one)
       * @type {String}
       */
      basePath() {
        const { startAngle, endAngle } = this

        return describePath(this.X_CENTER, this.Y_CENTER, this.RADIUS, startAngle, endAngle)
      },
     
      /**
       * d property of the gauge according to the value.
       * This gauge will hide a part of the base gauge
       * @type {String}
       */
      gaugePath() {
        const { endAngle, getAngle, tweenedValue } = this

        return describePath(this.X_CENTER, this.Y_CENTER, this.RADIUS, getAngle(tweenedValue), endAngle)
      },
      gaugePathCover() {
        const { endAngle, getAngle, tweenedValue } = this

        return describePathCover(this.mainRadius, this.innerRadius, this.X_CENTER, this.Y_CENTER, this.RADIUS, getAngle(tweenedValue), endAngle)
      },
      /**
       * Total angle of the gauge
       * @type {Number}
       */
      totalAngle() {
        const { startAngle, endAngle } = this

        return Math.abs(endAngle - startAngle)
      },
      /**
       * True if the gauge is a full circle
       * @type {Boolean}
       */
      isCircle() {
        return Math.abs(this.totalAngle) === 360
      },
      /**
       * True if the gaugeColor is an array
       * Result in displaying a gradient instead of a simple color
       * @type {Boolean}
       */
      hasGradient() {
        return Array.isArray(this.gaugeColor)
      },
      /**
       * Array of the path of each separator
       */
      separatorPaths() {
        const {
          separatorStep, getAngle, min, max, separatorThickness, isCircle,
        } = this

        if (separatorStep > 0) {
          const paths = []
          // If the gauge is a circle, this will add a separator at the start
          let i = isCircle ? min : min + separatorStep

          for (i; i < max; i += separatorStep) {
            const angle = getAngle(i)
            const halfAngle = separatorThickness / 2

            paths.push(describePath(this.X_CENTER, this.Y_CENTER, this.RADIUS + 2, angle - halfAngle, angle + halfAngle))
          }

          return paths
        }

        return null
      },
      /**
       * Array of line configuration for each scale
       */
      scaleLines() {
        const {
          scaleInterval, isCircle, min, max, getAngle, innerRadius,
        } = this

        if (scaleInterval > 0) {
          const lines = []
          // if gauge is a circle, remove the first scale
          let i = isCircle ? min + scaleInterval : min

          for (i; i < max + scaleInterval; i += scaleInterval) {
            const angle = getAngle(i)
            const startCoordinate = polarToCartesian(this.X_CENTER, this.Y_CENTER, innerRadius + 32, angle)
            const endCoordinate = polarToCartesian(this.X_CENTER, this.Y_CENTER, innerRadius + 30, angle)

            lines.push({
              xS: startCoordinate.x,
              yS: startCoordinate.y,
              xE: endCoordinate.x,
              yE: endCoordinate.y,
            })
          }

          return lines
        }

        return null
      },
    },
    watch: {
     
      /**
       * Watch the value and tween it to make an animation
       * If value < min, used value will be min
       * If value > max, used value will be max
       */
      value: {
        immediate: true,
        handler(next) {
          const { easing, tweenedValue, min, max, transitionDuration } = this
          let safeValue = next

          if (next < min) {
            safeValue = min
          }

          if (next > max) {
            safeValue = max
          }

          function animate() {
            if (TWEEN.update()) {
              requestAnimationFrame(animate)
            }
          }

          new TWEEN.Tween({ tweeningValue: tweenedValue })
            .to({ tweeningValue: safeValue }, transitionDuration)
            .easing(_get(TWEEN.Easing, easing))
            .onUpdate((object) => {
              this.tweenedValue = object.tweeningValue
            })
            .start()

            animate()
        },
      },
    },
    methods: {
      validateRaduis () {
        if (this.innerRadius < 0 || this.innerRadius > this.mainRadius) {
          console.warn(`GaugeChart - props "innerRadius" must be between 0 and ${this.RADIUS}`)
        }
        return true
      },       
      basePathM(startAngle, endAngle) {
        return describePath(this.X_CENTER, this.Y_CENTER, this.RADIUS, startAngle, endAngle)
      },
      basePoint(startAngle, endAngle) {
        return polarToCartesianPoint(this.X_CENTER, this.Y_CENTER, this.RADIUS, startAngle)
      },
      /**
       * Get an angle for a value
       * @param   {Number} value
       * @returns {Number} angle - in degree
       */
      getAngle(value) {
        const { min, max, startAngle, totalAngle } = this
        // Make sure not to divide by 0
        const totalValue = (max - min) || 1

        return ((value * totalAngle) / totalValue) + startAngle
      },
    },
  }
</script>

<style lang="css">
  .gauge {
    width: 100%;
    height: 100%;
    position: relative;
  }
  #taho-overlay {
    height: 240px;
    position: absolute;
    width: 240px;
    left: 0;
    top: 0;
  }
 .taho-overlay-onchart-digit {
    color: gray;
    font-size: 12px;
    position: absolute;
}
  #taho-overlay-main-digit {
    font-size: 48px;
    font-weight: 500;
    top: 50%;
        color: #000;
    left: 50%;
        transform: translate(-50%, -50%);
display: inline-block;
    position: absolute;
    text-align: center;
}

#taho-overlay-description {
    font-size: 18px;
    font-style: normal;
    font-weight: 400;
    color: #000;
    left: 20px;
    position: absolute;
    text-align: center;
    width: 100%;
    left: calc(50%);
    transform: translate(-50%, 0);
    bottom: 30px;
}
  
</style>