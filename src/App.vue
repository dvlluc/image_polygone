<template>
  <div id="app">
    <div>
      <v-stage
        ref="stage"
        :config="configKonva"
        style="position: relative, border: 1px solid red"
      >
        <v-layer ref="layer">
          <v-image
            :config="{
              image: image,
            }"
          />
          <v-line
            key="ssd"
            ref="polygone"
            :config="{
              points: polygoneCoordinates,
              closed: true,
              fill: '',
              stroke: 'red',
              draggable: true,
              dragBoundFunc: function (pos) {
                const selfRect = $refs.polygone.getStage().getSelfRect();

                const minX = -selfRect.x;
                const maxX =
                  $refs.stage.getStage().getWidth() -
                  (selfRect.x + selfRect.width);

                const minY = -selfRect.y;
                const maxY =
                  $refs.stage.getStage().getHeight() -
                  (selfRect.y + selfRect.height);

                return {
                  x: Math.min(maxX, Math.max(minX, pos.x)),
                  y: Math.min(maxY, Math.max(minY, pos.y)),
                };
              },
            }"
            @mousedown="mouseupAA"
            @dragmove="dragAndDropCoordinates"
          />
          <v-circle
            v-for="(item, index) in configCircleCoordinates"
            :config="item"
            :key="index"
            :ref="item.itemRef"
            @dragmove="moveCoordinate(index, $event)"
          ></v-circle>
        </v-layer>
      </v-stage>
    </div>
    <!-- //!!!!!! -->
    <h1 v-if="isCrossing">!!!</h1>
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
      confXY: {
        x: 0,
        y: 0,
        radius: 10,
        fill: "white",
      },
      configCircleCoordinates: [
        {
          itemRef: "circle0",
          x: 30,
          y: 30,
          radius: 20,
          fill: "brown",
          draggable: true,
          dragBoundFunc: null,
        },
        {
          itemRef: "circle1",
          x: 30,
          y: 480,
          radius: 20,
          fill: "green",
          draggable: true,
          dragBoundFunc: null,
        },
        {
          itemRef: "circle2",
          x: 480,
          y: 480,
          radius: 20,
          fill: "yellow",
          draggable: true,
          dragBoundFunc: null,
        },
        {
          itemRef: "circle3",
          x: 480,
          y: 30,
          radius: 20,
          fill: "blue",
          draggable: true,
          dragBoundFunc: null,
        },
      ],
      polygoneCoordinates: [30, 30, 30, 480, 480, 480, 480, 30],
      image: null,
      text: "0",

      positionPolygone: [30, 30, 30, 480, 480, 480, 480, 30],
      isCrossing: false,

      startingPolygonePoint: {
        x: 0,
        y: 0,
      },
    };
  },

  created() {
    const image = new window.Image();
    image.src =
      "https://static.nadzor.ua/uploads/images/Articles/kamera-nochnogo-videniya/8.jpg";
    image.onload = () => {
      this.image = image;
    };
    // ! прописываем ограничивающую полем логику для вершин полегона
    // this.configCircleCoordinates = this.configCircleCoordinates.map((el) => {
    //   el.dragBoundFunc = () =>
    //     function (pos) {
    //       return {
    //         x: Math.min(500, Math.max(0, pos.x)),
    //         y: Math.min(500, Math.max(0, pos.y)),
    //       };
    //     };
    //   return el;
    // });
  },
  methods: {
    changeX(index, event) {
      this.configCircleCoordinates[index].x = +event.target.value;
      //! для полегона надо учитывать смещение начала координат
      this.polygoneCoordinates[2 * index] =
        +event.target.value - this.startingPolygonePoint.x;
    },
    changeY(index, event) {
      this.configCircleCoordinates[index].y = +event.target.value;
      //! для полегона надо учитывать смещение начала координат
      this.polygoneCoordinates[2 * index + 1] =
        +event.target.value - this.startingPolygonePoint.y;
    },
    mouseupAA() {
      this.positionPolygone = [...this.$refs.polygone.getStage().getPoints()];
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
      p4 //проверка пересечения
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
      let x = event.target.attrs.x;
      let y = event.target.attrs.y;

      this.configCircleCoordinates[index].x = x;
      this.configCircleCoordinates[index].y = y;

      //! для полегона надо учитывать смещение начала координат
      this.polygoneCoordinates[2 * index] = x - this.startingPolygonePoint.x;
      this.polygoneCoordinates[2 * index + 1] =
        y - this.startingPolygonePoint.y;
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

      this.isCrossing = bypassClockwise | bypassCounterclockwise;

      if (this.isCrossing) {
        // перерэндер узлов
        // перерэндер полигона
      }
    },

    dragAndDropCoordinates(event) {
      this.startingPolygonePoint.x = event.target.attrs.x;
      this.startingPolygonePoint.y = event.target.attrs.y;

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
