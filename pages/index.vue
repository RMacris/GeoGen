<template>
  <div @dragover="onDrop($event)" id="container" class="container">
    <svg
      @wheel.prevent="zoom($event)"
      id="triangleWraper"
      viewBox=" 0 0 1408 740 "
      width="100%"
      height="100%"
    >
      <polygon
        id="triangle"
        points="0 0, 100 100 , 100 0"
        style="fill:#178a17;stroke:pink;stroke-width:1;vector-effect: non-scaling-stroke"
      />
      <rect
        id="adjRect"
        width="0"
        height="0"
        style="fill:rgb(167, 35, 35);stroke-width:1;stroke:rgb(0,0,0);vector-effect: non-scaling-stroke;"
      />
      <rect
        id="opRect"
        width="0"
        height="0"
        style="fill:rgb(189, 187, 37);stroke-width:1;stroke:rgb(0,0,0);vector-effect: non-scaling-stroke;"
      />
      <rect id="hipRect" width="0" height="0" />
      <text
        id="textAdj"
        x="20"
        y="35"
        fill="white"
        font-size-adjust="1"
        style="position:absolute;font-size: 10px;"
      >
        {{ sides.catAdj }}
      </text>
      <text
        id="textOp"
        x="40"
        y="35"
        fill="white"
        style="position:absolute;font-size: 8px;font-size-adjust: 0.8;"
        preserveAspectRatio="xMinYMin meet"
      >
        {{ sides.catOp }}
      </text>

      <text
        id="textHip"
        x="55"
        y="55"
        fill="white"
        font-size-adjust="0.1"
        style="position:absolute;font-size: 10px;"
        preserveAspectRatio="xMinYMin meet"
      >
        {{ sides.hip }}
      </text>
    </svg>

    <div
      @dragstart="dragImage($event)"
      @drag="moveCalculator($event)"
      @dragend="moveCalculator($event)"
      id="calculator"
      draggable="true"
    >
      <div>
        <h3>Calculadora</h3>
      </div>
      <div class="calc-inputs">
        <label for="catAdj">Cateto Adjacente </label>
        <input
          id="catAdj"
          type="text"
          v-model.number="sides.catAdj"
          value="0"
          preventOnFilter="false"
        />
        <label for="catOp">Cateto Oposto </label>
        <input
          id="catOp"
          type="text"
          v-model.number="sides.catOp"
          value="0"
          preventOnFilter="false"
        />
        <label for="hideRectLabel">
          Esconder Quadrados
          <input
            @click="hideRect($event)"
            type="checkbox"
            name=""
            id="hideRectLabel"
            class="hide-rect-checkbox"
            value=""
          />
        </label>
        <p v-if="!isFormValid" class="error">Digite apenas numeros</p>
        <p v-if="isNumToGreat" class="error">
          Tente manter os numeros menores que 3 digitos
        </p>
      </div>
      <div class="btn-actions">
        <button class="btn --calc" @click="resolveCalculation()">
          Calcular
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      trianglePosX: 0,
      trianglePosY: 0,
      sides: {
        catOp: 12,
        catAdj: 5,
        hip: 0
      },

      scale: 1,
      dragStartOffsetX: 0,
      dragStartOffsetY: 0
    }
  },
  computed: {
    isFormValid() {
      return (
        typeof this.sides.catAdj === 'number' &&
        typeof this.sides.catOp === 'number'
      )
    },
    isNumToGreat() {
      return this.sides.catOp > 100 || this.sides.catAdj > 100
    }
  },
  mounted() {
    this.setSvgInitialViewBoxVal()
    this.drawTriangle()
  },
  methods: {
    async requestHipCalc() {
      try {
        const url = 'https://atlas-231814.appspot.com/calcula'
        const triangleParams = {
          cat_op: parseInt(this.sides.catOp),
          cat_adj: parseInt(this.sides.catAdj)
        }
        const result = await this.$axios.$post(url, triangleParams)
        console.warn(result)
        this.sides.hip = parseFloat(result.hip).toFixed(2)
      } catch (error) {
        await alert('Ops, ocorreu um erro, tente reiniciar a pagina')
      }
    },

    dragImage(event) {
      /** desativa imagem fantasma enquanto arrasta calculadora */
      this.dragStartOffsetX = event.offsetX
      this.dragStartOffsetY = event.offsetY
      const img = new Image()
      img.src = ''
      event.dataTransfer.setDragImage(img, 0, 0)
    },
    moveCalculator(event) {
      // movimenta calculadora a partir do ponto de click inicial
      const calculator = document.getElementById('calculator')

      calculator.style.top = `${event.clientY - this.dragStartOffsetY}px`
      calculator.style.left = `${event.clientX - this.dragStartOffsetX}px`
      // ensure full controll
      event.preventDefault()
    },
    onDrop(event) {
      // previne falha ao soltar
      event.preventDefault()
    },
    setSvgInitialViewBoxVal() {
      // seta valor inicial par viewBox = largura e altura do parent
      const container = this.getContainerSize()
      const svg = document.getElementById('triangleWraper')
      svg.setAttribute('viewBox', `0 0 ${container.width} ${container.height}`)
    },
    getContainerSize() {
      return document.getElementById('container').getBoundingClientRect()
    },
    setMaxSize(catOp, catAdj) {
      // **  previne valores negativos * /

      // const container = this.getContainerSize()
      // catOp = (catOp / container.width) * 100
      // catAdj = (catAdj / container.height) * 100
      // if (catOp > container.width) {
      //   catOp = container.width
      // }
      if (catOp < 0) {
        catOp = 0
      }
      // if (catAdj > container.height) {
      //   catAdj = container.height
      // }
      if (catAdj < 0) {
        catAdj = 0
      }

      return { catOp, catAdj }
    },
    drawTriangle() {
      // ** Desenha o triangulo na tela */
      if (!this.isFormValid) return
      const poly = document.getElementById('triangle')
      const op = this.sides.catOp
      const adj = this.sides.catAdj
      const { catOp, catAdj } = this.setMaxSize(op, adj)
      poly.setAttribute('points', `0 0, 0 ${catAdj}, ${catOp} ${catAdj}`)

      this.centerTriangle()
      // centraliza o texto relativamente ao triangulo
      this.setTextPosition(this.trianglePosX, this.trianglePosY)
      // centraliza o retangulo relativamente ao triangulo
      this.drawRect(this.trianglePosX, this.trianglePosY)
    },
    centerTriangle() {
      // ** centraliza o triangulo dentro do seu envelope */
      const poly = document.getElementById('triangle')
      const svg = document.getElementById('triangleWraper')
      const op = this.sides.catOp
      const adj = this.sides.catAdj
      const { catOp, catAdj } = this.setMaxSize(op, adj)
      // returna os valores da viewBox como array
      const viewBoxAttributes = svg.getAttribute('viewBox').split(' ')
      const viewBoxWidth = parseFloat(viewBoxAttributes[2])
      const viewBoxHeight = parseFloat(viewBoxAttributes[3])
      /*
        centraliza o triangulo subtraindo metade da sua largura e altura 
       */
      const triangleCenterX = viewBoxWidth / 2 - catOp / 2
      const triangleCenterY = viewBoxHeight / 2 - catAdj / 2
      this.trianglePosX = triangleCenterX
      this.trianglePosY = triangleCenterY
      // poly.setAttribute('points', `0 0, 0 ${catAdj}, ${catOp} ${catAdj}`)
      poly.setAttribute(
        'transform',
        `translate(${triangleCenterX},${triangleCenterY})`
      )
    },

    async resolveCalculation() {
      if (this.isFormValid) {
        await this.requestHipCalc()
        await this.drawTriangle()
      }
    },
    zoom(event) {
      const poly = document.getElementById('triangle')

      const zoomRate = event.deltaY
      const svg = document.getElementById('triangleWraper')
      const container = this.getContainerSize()
      const viewBoxAttributes = svg.getAttribute('viewBox').split(' ')
      const viewBoxWidth = parseFloat(viewBoxAttributes[2]) + zoomRate
      const viewBoxHeight = parseFloat(viewBoxAttributes[3]) + zoomRate
      console.log(container.width, container.height)
      const maxZoomWidth = Math.min(
        Math.max(container.width / 24, viewBoxWidth),
        container.width * 8
      )
      const maxZoomHeight = Math.min(
        Math.max(container.height / 24, viewBoxHeight),
        container.height * 8
      )
      svg.setAttribute('viewBox', `0 0 ${maxZoomWidth} ${maxZoomHeight}`)
      poly.setAttribute(
        'transform',
        `translate(${this.trianglePosX},${this.trianglePosY})`
      )
      this.drawTriangle()
    },

    setTextPosition(triangleCenterX, triangleCenterY) {
      /* seta posição texto em relação ao centro do triangulo */
      const textAdj = document.getElementById('textAdj')
      const textOp = document.getElementById('textOp')
      const textHip = document.getElementById('textHip')

      const vect = this.getTriangleInfo().vectors
      const points = this.getTriangleInfo().points

      textAdj.setAttribute('x', vect.adjVector.x * 0.5 + triangleCenterX)
      textAdj.setAttribute('y', vect.adjVector.y * 0.75 + triangleCenterY)
      textOp.setAttribute('x', vect.opVector.x * 0.5 + triangleCenterX)
      textOp.setAttribute(
        'y',
        points.third.y * 2 - points.second.y + triangleCenterY
      )
      textHip.setAttribute('x', vect.hipVector.x * 0.5 + triangleCenterX)
      textHip.setAttribute('y', vect.hipVector.y * 0.5 + triangleCenterY)
      this.adjustFontSize()
    },
    adjustFontSize() {
      const texts = document.querySelectorAll('text')
      let fontsize = 10 // em porcentagem
      texts.forEach((textEl) => {
        if (this.sides.catOp < 10 || this.sides.catAdj < 10) fontsize = 10
        else if (this.sides.catOp < 10 || this.sides.catAdj < 10) fontsize = 200
        else
          fontsize =
            this.sides.catOp > this.sides.catAdj
              ? this.sides.catOp
              : this.sides.catAdj

        if (fontsize > 100) fontsize = 100
        textEl.style.fontSize = `${fontsize}%`
      })
    },
    getTriangleInfo() {
      const poly = document.getElementById('triangle')
      const coordinates = poly.getAttribute('points').split(/[\n ,]{1,}/gi)
      const points = {
        first: {
          x: coordinates[0],
          y: coordinates[1]
        },
        second: {
          x: coordinates[2],
          y: coordinates[3]
        },
        third: {
          x: coordinates[4],
          y: coordinates[5]
        }
      }
      const vectors = {
        adjVector: {
          x: points.second.x - points.first.x, // * 0.5 + triangleCenterX, // move text to the left 20px
          y: points.second.y - points.first.y // * 0.75 + triangleCenterY
        },
        opVector: {
          x: points.third.x - points.second.x, // * 0.5 + triangleCenterX, // move text left
          y: points.third.y - points.second.y // + triangleCenterY // move text down 20px
        },
        hipVector: {
          x: points.third.x - points.first.x, // * 0.5 + triangleCenterX, // move text to the left 20px
          y: points.third.y - points.first.y // * 0.5 + triangleCenterY
          // move text down 20px
        }
      }

      return { vectors, points }
    },
    setRectPosition(triangleCenterX, triangleCenterY) {
      /* posição do retangulo em relação ao centro do triangulo */
      const adjRect = document.getElementById('adjRect')
      const opRect = document.getElementById('opRect')
      const hipRect = document.getElementById('hipRect')

      const points = this.getTriangleInfo().points

      adjRect.setAttribute('x', points.first.x - this.sides.catAdj)
      adjRect.setAttribute('y', points.first.y)
      opRect.setAttribute('x', points.second.x)
      opRect.setAttribute('y', points.second.y)

      //! translate rect relative to triangle center
      adjRect.setAttribute(
        'transform',
        `translate(${triangleCenterX},${triangleCenterY})`
      )
      opRect.setAttribute(
        'transform',
        `translate(${triangleCenterX},${triangleCenterY})`
      )
      // calcula o angula de inclinação da hipotenusa
      let angle = Math.atan2(this.sides.catAdj, this.sides.catOp)
      angle = (angle * 180) / Math.PI

      hipRect.setAttribute(
        'transform',
        `translate(${triangleCenterX},${triangleCenterY})
        rotate(${angle - 90})
        `
      )
    },
    setRectWidth() {
      const adjRect = document.getElementById('adjRect')
      const opRect = document.getElementById('opRect')
      const hipRect = document.getElementById('hipRect')

      adjRect.setAttribute('width', `${this.sides.catAdj}`)
      adjRect.setAttribute('height', `${this.sides.catAdj}`)
      opRect.setAttribute('width', `${this.sides.catOp}`)
      opRect.setAttribute('height', `${this.sides.catOp}`)
      hipRect.setAttribute('width', `${this.sides.hip}`)
      hipRect.setAttribute('height', `${this.sides.hip}`)
    },
    drawRect(triangleCenterX, triangleCenterY) {
      this.setRectPosition(triangleCenterX, triangleCenterY)
      this.setRectWidth()
    },
    hideRect(event) {
      const adjRect = document.getElementById('adjRect')
      const opRect = document.getElementById('opRect')
      const hipRect = document.getElementById('hipRect')
      adjRect.style.visibility = event.target.checked ? 'hidden' : 'visible'
      opRect.style.visibility = event.target.checked ? 'hidden' : 'visible'
      hipRect.style.visibility = event.target.checked ? 'hidden' : 'visible'
    }
  }
}
</script>

<style>
h3 {
  line-height: 48px;
  padding: 0 24px;
  background-color: rgb(35, 35, 35);
  color: white;
  text-align: center;
}
#hipRect {
  fill: rgb(14, 93, 181);
  stroke-width: 1;
  stroke: rgb(0, 0, 0);
  vector-effect: non-scaling-stroke;
  fill: rgb(22, 92, 170);
}
#triangleWraper {
  margin: 0 auto;
  align-self: center;
}
#triangleWraper text {
  position: absolute;
  color: white;
}
#calculator {
  position: absolute;
  min-width: 180px;
  width: 25%;
  max-width: 230px;
  background-color: white;
  box-shadow: 0 0 8px 2px rgb(10, 10, 10);
  border-radius: 2px;
}
#hideRectLabel {
  box-shadow: none;
}
.calc-inputs {
  width: 100%;
  display: flex;
  flex-direction: column;
  padding: 12px 24px 0 24px;
}
.calc-inputs > * {
  text-align: start;
  width: 100%;
}
.calc-inputs input {
  border-radius: 4px;
  border-style: none;
  padding: 8px;
  box-shadow: 0 0 4px 0 rgb(160, 160, 160) inset;
}
.btn-actions {
  padding: 12px 24px;
}
.btn {
  width: 100%;
  border: none;
  border-radius: 2px;
  padding: 8px;
  border: 1px solid rgb(35, 35, 35);
  background-color: rgb(35, 35, 35);
  color: white;
  outline: none;
}
.btn:hover {
  background-color: rgb(45, 45, 45);
}

.btn:active {
  background-color: rgb(65, 65, 65);
}

.container {
  position: relative;
  margin: 0 auto;
  display: flex;
  width: 100%;
  height: 100%;
  max-height: 100%;
  overflow: hidden;
}
.container {
  background-color: transparent;
  background-image: linear-gradient(
      0deg,
      transparent 24%,
      rgba(255, 255, 255, 0.05) 25%,
      rgba(255, 255, 255, 0.05) 26%,
      transparent 27%,
      transparent 74%,
      rgba(255, 255, 255, 0.05) 75%,
      rgba(255, 255, 255, 0.05) 76%,
      transparent 77%,
      transparent
    ),
    linear-gradient(
      90deg,
      transparent 24%,
      rgba(255, 255, 255, 0.05) 25%,
      rgba(255, 255, 255, 0.05) 26%,
      transparent 27%,
      transparent 74%,
      rgba(255, 255, 255, 0.05) 75%,
      rgba(255, 255, 255, 0.05) 76%,
      transparent 77%,
      transparent
    );
  height: 100%;
  background-size: 50px 50px;
  animation-name: zoomSlow;
  animation-duration: 4s;
}
.error {
  color: red;
  background-color: rgb(255, 220, 220);
  padding: 4px 24px;
  font-size: 0.8rem;
}
@media (max-width: 1008px) {
}
</style>
