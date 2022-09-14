<template>
  <div id="app">
    <div>
      <v-stage
        ref="stage"
        :config="configKonva"
        style="position: relative, border: 1px solid red"
      >
        <v-layer ref="layer">
          <v-group ref="group">
            <v-image
              :config="{
                image: image,
              }"
            />
            <v-line
              ref="polygone"
              :config="{
                points: polygoneCoordinates,
                closed: true,
                fill: '',
                stroke: 'red',
                draggable: true,
              }"
              @mousedown="mouseupAA"
              @dragmove="dragAndDropCoordinates"
            />

            <v-circle
              v-for="(item, index) in configCircleCoordinates"
              :config="item"
              :key="index"
              @dragmove="moveCoordinate(index, $event)"
            ></v-circle>
          </v-group>
        </v-layer>
      </v-stage>
    </div>
    <div
      v-for="(item, index) in configCircleCoordinates"
      :style="{ background: `${item.fill}`, color: 'white', width: '500px' }"
      :key="index + index"
    >
      {{ `X${index}` }}
      <input
        :value="configCircleCoordinates[index].x"
        @input="changeX(index, $event)"
        type="number"
      />
      {{ `Y${index}` }}
      <input
        :value="configCircleCoordinates[index].y"
        @input="changeY(index, $event)"
        type="number"
      />
    </div>
  </div>
</template>

<script>
// eslint-disable-next-line no-unused-vars
const width = window.innerWidth;

// eslint-disable-next-line no-unused-vars
const height = window.innerHeight;

export default {
  data() {
    return {
      configKonva: {
        width: 500,
        height: 500,
      },
      configCircleCoordinates: [
        {
          x: 30,
          y: 30,
          radius: 20,
          fill: "brown",
          draggable: true,
        },
        {
          x: 30,
          y: 480,
          radius: 20,
          fill: "green",
          draggable: true,
        },
        {
          x: 480,
          y: 480,
          radius: 20,
          fill: "yellow",
          draggable: true,
        },
        {
          x: 480,
          y: 30,
          radius: 20,
          fill: "blue",
          draggable: true,
        },
      ],
      polygoneCoordinates: [30, 30, 30, 480, 480, 480, 480, 30],

      image: null,

      positionPolygone: [],
      positionPolygoneCursor: {},

      isCrossing: false,
    };
  },

  created() {
    const image = new window.Image();
    image.src =
      "https://static.nadzor.ua/uploads/images/Articles/kamera-nochnogo-videniya/8.jpg";
    image.onload = () => {
      this.image = image;
    };
  },

  methods: {
    changeX(index, event) {
      this.configCircleCoordinates[index].x = event.target.value;
      this.polygoneCoordinates[2 * index] = event.target.value;
    },
    changeY(index, event) {
      this.configCircleCoordinates[index].y = event.target.value;
      this.polygoneCoordinates[2 * index + 1] = event.target.value;
    },
    mouseupAA() {
      this.positionPolygone = this.polygoneCoordinates.slice(0);
      const mousePos = this.$refs.stage.getNode().getPointerPosition();
      this.positionPolygoneCursor.x = Math.floor(mousePos.x);
      this.positionPolygoneCursor.y = Math.floor(mousePos.y);
    },

    VEK(
      ax,
      ay,
      bx,
      by //векторное произведение
    ) {
      return ax * by - bx * ay;
    },

    crossingCheck(
      p1,
      p2,
      p3,
      p4 //проверка пересечения 2-x не смежных сторон
    ) {
      let v1, v2, v3, v4;

      v1 = this.VEK(p4.x - p3.x, p4.y - p3.y, p1.x - p3.x, p1.y - p3.y);
      v2 = this.VEK(p4.x - p3.x, p4.y - p3.y, p2.x - p3.x, p2.y - p3.y);
      v3 = this.VEK(p2.x - p1.x, p2.y - p1.y, p3.x - p1.x, p3.y - p1.y);
      v4 = this.VEK(p2.x - p1.x, p2.y - p1.y, p4.x - p1.x, p4.y - p1.y);
      if (v1 * v2 < 0 && v3 * v4 < 0) return true;
      else return false;
    },

    moveCoordinate(index, event) {
      let x = Math.floor(event.target.attrs.x);
      let y = Math.floor(event.target.attrs.y);

      this.configCircleCoordinates[index].x = x;
      this.configCircleCoordinates[index].y = y;

      this.polygoneCoordinates[2 * index] = x;
      this.polygoneCoordinates[2 * index + 1] = y;
      const points = this.configCircleCoordinates;

      // обход по часовой стрелке
      const bypassClockwise = this.crossingCheck(
        points[0],
        points[3],
        points[2],
        points[1]
      );

      // обход против часовой стрелке
      const bypassCounterclockwise = this.crossingCheck(
        points[0],
        points[1],
        points[2],
        points[3]
      );

      // проверка пересечений в полигоне
      this.isCrossing = bypassClockwise | bypassCounterclockwise;

      // if (this.isCrossing) {
      //   // перерэндер узлов
      //   [this.configCircleCoordinates[0], this.configCircleCoordinates[2]] = [
      //     this.configCircleCoordinates[2],
      //     this.configCircleCoordinates[0],
      //   ];
      //   // перерэндер полигона
      //   [this.polygoneCoordinates[0], this.polygoneCoordinates[4]] = [
      //     this.polygoneCoordinates[4],
      //     this.polygoneCoordinates[0],
      //   ];
      //   [this.polygoneCoordinates[1], this.polygoneCoordinates[5]] = [
      //     this.polygoneCoordinates[5],
      //     this.polygoneCoordinates[1],
      //   ];

      // }
    },

    dragAndDropCoordinates(event) {
      this.configCircleCoordinates[0].x =
        this.positionPolygone[0] + event.target.attrs.x;
      this.configCircleCoordinates[0].y =
        this.positionPolygone[1] + event.target.attrs.y;

      this.configCircleCoordinates[1].x =
        this.positionPolygone[2] + event.target.attrs.x;
      this.configCircleCoordinates[1].y =
        this.positionPolygone[3] + event.target.attrs.y;

      this.configCircleCoordinates[2].x =
        this.positionPolygone[4] + event.target.attrs.x;
      this.configCircleCoordinates[2].y =
        this.positionPolygone[5] + event.target.attrs.y;

      this.configCircleCoordinates[3].x =
        this.positionPolygone[6] + event.target.attrs.x;
      this.configCircleCoordinates[3].y =
        this.positionPolygone[7] + event.target.attrs.y;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
