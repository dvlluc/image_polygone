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
            ref="polygone"
            :config="{
              points: polygoneCoordinates,
              closed: true,
              fill: '',
              stroke: 'red',
              draggable: true,
              dragBoundFunc: (pos) => {
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
            @dragstart="saveCoordinatesBeforeDrug"
            @dragend="checkCoordinatesAfterDrug"
            @dragmove="moveCoordinate(index, $event)"
            @mouseover="makeMainPoint(index)"
          ></v-circle>
        </v-layer>
      </v-stage>
    </div>
    <h1>{{ startingPolygonePoint.x }} : {{ startingPolygonePoint.y }}</h1>
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
// const width = window.innerWidth;
const width = 500;
// eslint-disable-next-line no-unused-vars
// const height = window.innerHeight;
const height = 500;
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
          dragBoundFunc(pos) {
            return {
              x: Math.min(width, Math.max(0, pos.x)),
              y: Math.min(height, Math.max(0, pos.y)),
            };
          },
        },
        {
          x: 30,
          y: 480,
          radius: 20,
          fill: "green",
          draggable: true,
          dragBoundFunc(pos) {
            return {
              x: Math.min(width, Math.max(0, pos.x)),
              y: Math.min(height, Math.max(0, pos.y)),
            };
          },
        },
        {
          x: 480,
          y: 480,
          radius: 20,
          fill: "yellow",
          draggable: true,
          dragBoundFunc(pos) {
            return {
              x: Math.min(width, Math.max(0, pos.x)),
              y: Math.min(height, Math.max(0, pos.y)),
            };
          },
        },
        {
          x: 480,
          y: 30,
          radius: 20,
          fill: "blue",
          draggable: true,
          dragBoundFunc(pos) {
            return {
              x: Math.min(width, Math.max(0, pos.x)),
              y: Math.min(height, Math.max(0, pos.y)),
            };
          },
        },
      ],
      polygoneCoordinates: [30, 30, 30, 480, 480, 480, 480, 30],
      image: null,
      text: "0",
      positionPolygone: [30, 30, 30, 480, 480, 480, 480, 30],
      crossingNumber: 0,
      // стартовая позиция поля детекции при Drad and Drop
      startingPolygonePoint: {
        x: 0,
        y: 0,
      },
      // стартовая позиция точки детекции при Drad and Drop
      startingCirclePoint: {
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
  },
  methods: {
    makeMainPoint(index) {
      if (index === 1) {
        const b = [
          this.configCircleCoordinates[1],
          this.configCircleCoordinates[2],
          this.configCircleCoordinates[3],
          this.configCircleCoordinates[0],
        ];
        this.configCircleCoordinates = [...b];
      } else if (index === 2) {
        const b = [
          this.configCircleCoordinates[2],
          this.configCircleCoordinates[3],
          this.configCircleCoordinates[0],
          this.configCircleCoordinates[1],
        ];
        this.configCircleCoordinates = [...b];
      } else if (index === 3) {
        const b = [
          this.configCircleCoordinates[3],
          this.configCircleCoordinates[0],
          this.configCircleCoordinates[1],
          this.configCircleCoordinates[2],
        ];
        this.configCircleCoordinates = [...b];
      }
      this.polygoneCoordinates = this.configCircleCoordinates.reduce(
        (acc, curr) => {
          return [
            ...acc,
            +curr.x - this.startingPolygonePoint.x,
            +curr.y - this.startingPolygonePoint.y,
          ];
        },
        []
      );
    },

    changeX(index, event) {
      this.configCircleCoordinates[index].x = +event.target.value;
      //! для полигона надо учитывать смещение начала координат
      this.polygoneCoordinates[2 * index] =
        +event.target.value - this.startingPolygonePoint.x;
    },
    changeY(index, event) {
      this.configCircleCoordinates[index].y = +event.target.value;
      //! для полигона надо учитывать смещение начала координат
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
      let v1;
      let v2;
      let v3;
      let v4;
      v1 = this.VEK(p4.x - p3.x, p4.y - p3.y, p1.x - p3.x, p1.y - p3.y);
      v2 = this.VEK(p4.x - p3.x, p4.y - p3.y, p2.x - p3.x, p2.y - p3.y);
      v3 = this.VEK(p2.x - p1.x, p2.y - p1.y, p3.x - p1.x, p3.y - p1.y);
      v4 = this.VEK(p2.x - p1.x, p2.y - p1.y, p4.x - p1.x, p4.y - p1.y);
      return (v1 * v2 < 0 && v3 * v4 < 0) || (v1 * v4 < 0 && v2 * v3 < 0);
    },

    saveCoordinatesBeforeDrug() {
      const startPoint = { ...this.configCircleCoordinates[0] };
      this.startingCirclePoint = {
        x: startPoint.x,
        y: startPoint.y,
      };
    },

    checkCoordinatesAfterDrug() {
      const xList = [];
      const yList = [];

      this.configCircleCoordinates.forEach((el) => {
        xList.push(el.x);
        yList.push(el.y);
      });

      const maxX = this.getMaxOfArray(xList);
      const maxY = this.getMaxOfArray(yList);
      const minX = this.getMinOfArray(xList);
      const minY = this.getMinOfArray(yList);

      const checkX = ((maxX - minX) / width) * 100;
      const checkY = ((maxY - minY) / height) * 100;

      if (!(checkX < 10 || checkY < 10)) {
        return;
      }
      this.configCircleCoordinates[0].x = this.startingCirclePoint.x;
      this.configCircleCoordinates[0].y = this.startingCirclePoint.y;

      this.polygoneCoordinates = this.configCircleCoordinates.reduce(
        (acc, curr) => {
          return [
            ...acc,
            +curr.x - this.startingPolygonePoint.x,
            +curr.y - this.startingPolygonePoint.y,
          ];
        },
        []
      );
    },
    getMaxOfArray(numArray) {
      return Math.max.apply(null, numArray);
    },
    getMinOfArray(numArray) {
      return Math.min.apply(null, numArray);
    },

    moveCoordinate(index, event) {
      const { x, y } = event.target.attrs;
      this.currentCoordinate = { x, y };
      this.configCircleCoordinates[index].x = x;
      this.configCircleCoordinates[index].y = y;
      //! для полигона надо учитывать смещение начала координат
      this.polygoneCoordinates[2 * index] = x - this.startingPolygonePoint.x;
      this.polygoneCoordinates[2 * index + 1] =
        y - this.startingPolygonePoint.y;
      const p = this.configCircleCoordinates;
      let isCrossing = this.crossingCheck(p[0], p[1], p[2], p[3]);
      if (isCrossing) {
        if (!this.crossingCheck(p[0], p[2], p[1], p[3])) {
          const a = [p[0], p[2], p[1], p[3]];
          this.configCircleCoordinates = [...a];
        } else if (!this.crossingCheck(p[0], p[2], p[3], p[1])) {
          const a = [p[0], p[2], p[3], p[1]];
          this.configCircleCoordinates = [...a];
        }
      }
      this.polygoneCoordinates = this.configCircleCoordinates.reduce(
        (acc, curr) => {
          return [
            ...acc,
            +curr.x - this.startingPolygonePoint.x,
            +curr.y - this.startingPolygonePoint.y,
          ];
        },
        []
      );
      isCrossing = false;
    },
    dragAndDropCoordinates2(event) {
      const { x, y } = event.target.attrs;
      this.startingPolygonePoint.x = x;
      this.startingPolygonePoint.y = y;

      this.configCircleCoordinates[0].x = this.positionPolygone[0] + x;
      this.configCircleCoordinates[0].y = this.positionPolygone[1] + y;
      this.configCircleCoordinates[1].x = this.positionPolygone[2] + x;
      this.configCircleCoordinates[1].y = this.positionPolygone[3] + y;
      this.configCircleCoordinates[2].x = this.positionPolygone[4] + x;
      this.configCircleCoordinates[2].y = this.positionPolygone[5] + y;
      this.configCircleCoordinates[3].x = this.positionPolygone[6] + x;
      this.configCircleCoordinates[3].y = this.positionPolygone[7] + y;
    },
    dragAndDropCoordinates(event) {
      const { x, y } = event.target.attrs;
      this.startingPolygonePoint.x = x;
      this.startingPolygonePoint.y = y;

      this.configCircleCoordinates.forEach((_, index) => {
        this.configCircleCoordinates[index] = Object.assign(
          this.configCircleCoordinates[index],
          {
            x: this.positionPolygone[2 * index] + x,
            y: this.positionPolygone[2 * index + 1] + y,
          }
        );
      });
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
