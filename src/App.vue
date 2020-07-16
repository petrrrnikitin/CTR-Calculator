<template>
  <div id="app">
    <div class="container-fluid min-vh-100">
      <div class="row mb-5">
        <div class="offset-1 col-10 mt-5 text-muted">
          <h1 class="mb-5 text-center">Калькулятор расчета CTR и необходимой выборки для проверки гипотез</h1>
          Классическая формула расчета CTR выглядит как соотношение кликов к показам. К сожалению, она верна только при достаточном количестве показов рекламы. Важная задача
          маркетолога &mdash; не поторопиться с принятием решения на счет того, насколько хорошо или плохо отрабатывает его креатив. Наш калькулятор рассчитывает диапазон CTR (от и
          до) при текущем количестве кликов и показов, а также необходимую выборку по количеству показов, чтобы определить, насколько Ваш реальный CTR может соответствовать
          желаемому (например, среднему по значению Яндекс.Директа)

          <p class="mt-3">Для расчетов мы сначала вычисляли выборку, а потом с помощью формулы Уилсона определяли диапазон CTR для разной вероятности.</p>
        </div>
        <div id="form" :class="[showTable ? 'moveInLeft' : '']">
          <form @submit.prevent="calc" class="form" action="">
            <div class="form-group">
              <label for="showing">Количество показов</label>
              <input @change="calc" v-model.number="showing" type="number" min="0" step="1" class="form-control" id="showing" placeholder="Введите количество показов" />
            </div>
            <div class="form-group">
              <label for="clicks">Количество кликов</label>
              <input @change="calc" v-model.number="clicks" type="number" min="0" step="1" class="form-control" id="clicks" placeholder="Введите количество кликов" />
            </div>
            <div class="form-group">
              <label for="wishful_ctr">Желаемый CTR в %</label>
              <input @change="calc" v-model.number="wishfulCtr" min="0" max="100" type="number" class="form-control" id="wishful_ctr" placeholder="Введите желаемый CTR" />
            </div>
            <div class="form-group">
              <label for="fault">Погрешность в % (допустимое отклонение)</label>
              <input @change="calc" v-model.number="fault" min="0.1" max="100" step="0.1" type="number" class="form-control" id="fault" placeholder="Введите погрешность" />
            </div>

            <button type="submit" id="btn" class="btn-info btn">Рассчитать</button>
            <div class="start mt-3"></div>
          </form>
        </div>
        <div id="table" :class="[showTable ? 'fadeIn' : 'd-none']" class="col-7 offset-5 my-5">
          <table class="table mt-2 w-75 mx-auto table-bordered text-center">
            <thead>
              <tr>
                <td colspan="3" scope="col">
                  <span class="d-block text-center">На данный момент Ваш CTR находится в диапазоне</span>
                </td>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>с точностью 68%</td>
                <td>с точностью 95%</td>
                <td>с точностью 99.7%</td>
              </tr>
              <tr>
                <td>
                  от <span class="range_ctr_min1">{{ (ctrArray[0][0] * 100).toFixed(2) + "%" }}</span> до
                  <span class="range_ctr_max1">{{ (ctrArray[0][1] * 100).toFixed(2) + "%" }}</span>
                </td>
                <td>
                  от <span class="range_ctr_min2">{{ (ctrArray[1][0] * 100).toFixed(2) + "%" }}</span> до
                  <span class="range_ctr_max2">{{ (ctrArray[1][1] * 100).toFixed(2) + "%" }}</span>
                </td>
                <td>
                  от <span class="range_ctr_min3">{{ (ctrArray[2][0] * 100).toFixed(2) + "%" }}</span> до
                  <span class="range_ctr_max3">{{ (ctrArray[2][1] * 100).toFixed(2) + "%" }}</span>
                </td>
              </tr>
              <tr>
                <td colspan="3" scope="col">
                  <span class="d-block text-center">Чтобы проверить насколько Ваш CTR будет соответствовать желаемому осталось показов:</span>
                </td>
              </tr>
              <tr>
                <td>с точностью 68%</td>
                <td>с точностью 95%</td>
                <td>с точностью 99.7%</td>
              </tr>
              <tr>
                <td class="showing_ad_to_wishful_ctr1 text-center">
                  {{ showArray[0] ? "" : "Показов проведено достаточно, эксперимент закончен" }}
                  <span v-if="needToShowArray[0][0]" class="showing_ad_to_wishful_ctr_min1"> {{ needToShowArray[0][0] }}</span>
                  <span v-if="needToShowArray[0][1]" class="showing_ad_to_wishful_ctr_max1"> {{ needToShowArray[0][1] }}</span>
                </td>
                <td class="showing_ad_to_wishful_ctr2 text-center">
                  {{ showArray[1] ? "" : "Показов проведено достаточно, эксперимент закончен" }}
                  <span v-if="needToShowArray[1][0]" class="showing_ad_to_wishful_ctr_min2"> {{ needToShowArray[1][0] }}</span>
                  <span v-if="needToShowArray[1][1]" class="showing_ad_to_wishful_ctr_max2"> {{ needToShowArray[1][1] }}</span>
                </td>
                <td class="showing_ad_to_wishful_ctr3 text-center">
                  {{ showArray[2] ? "" : "Показов проведено достаточно, эксперимент закончен" }}
                  <span v-if="needToShowArray[2][0]" class="showing_ad_to_wishful_ctr_min3"> {{ needToShowArray[2][0] }}</span>
                  <span v-if="needToShowArray[2][1]" class="showing_ad_to_wishful_ctr_max3"> {{ needToShowArray[2][1] }}</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    <div v-if="showTable">
        <div v-if="showArray[0]" class="bg-white mx-auto" id="chart_div1">
        <GChart type="BarChart" :data="[chartData[0][0], chartData[0][1], wishArray]" :options="options1" />
      </div>
      <div v-else class="bg-white mx-auto text-center text-info text-big">
        {{ checkArray[0] ? "С точностью 68% - эксперимент закончен. Цель достигнута." : "С точностью 68% - эксперимент закончен. Цель не достигнута." }}
      </div>

      <div v-if="showArray[1]" class="bg-white mx-auto" id="chart_div2">
        <GChart type="BarChart" :data="[chartData[1][0], chartData[1][1], wishArray]" :options="options2" />
      </div>
      <div v-else class="bg-white mx-auto text-center text-info text-big">
        {{ checkArray[1] ? "С точностью 95% - эксперимент закончен. Цель достигнута." : "С точностью 95% - эксперимент закончен. Цель не достигнута." }}
      </div>

      <div v-if="showArray[2]" class="bg-white mx-auto" id="chart_div3">
        <GChart type="BarChart" :data="[chartData[2][0], chartData[2][1], wishArray]" :options="options3" />
      </div>
      <div v-else class="bg-white mx-auto text-center text-info text-big">
        {{ checkArray[2] ? "С точностью 99.7% - эксперимент закончен. Цель достигнута." : "С точностью 99.7% - эксперимент закончен. Цель не достигнута." }}
      </div>
    </div>
    </div>
  </div>
</template>
<script>
import { GChart } from "vue-google-charts";
export default {
  name: "App",
  components: {
    GChart,
  },
  data() {
    return {
      showing: null,
      clicks: null,
      fault: null,
      wishfulCtr: null,
      showTable: false,
      realValue: "",
      wishfulCtrValueMin: null,
      wishfulCtrValueMax: null,
      // массив выборки
      selectionArray: [],
      // массив для отображения данных или вывода на основе расчетов
      showArray: [],
      //массив желаемого ctr для отображения на графике
      wishArray: [],
      //сигма для разных точностей
      Sigma: [1, 1.96, 3],
      //массив реального диапазона
      ctrArray: [[], [], []],
      checkArray: [],
      needToShowArray: [[], [], []],
      chartData: [
        [
          [
            "CTR",
            "Реальный CTR",
            "CTR в выборке",
            {
              type: "string",
              role: "tooltip",
            },
            {
              type: "string",
              role: "style",
            },
          ],
        ],
        [
          [
            "CTR",
            "Реальный CTR",
            "CTR в выборке",
            {
              type: "string",
              role: "tooltip",
            },
            {
              type: "string",
              role: "style",
            },
          ],
        ],
        [
          [
            "CTR",
            "Реальный CTR",
            "CTR в выборке",
            {
              type: "string",
              role: "tooltip",
            },
            {
              type: "string",
              role: "style",
            },
          ],
        ],
      ],
      options1: {
        legend: "none",
        title: "Диапазоны CTR с точностью 68%",
        chartArea: {
          width: "50%",
        },
        isStacked: true,
        hAxis: {
          title: "CTR",
          minValue: 0,
        },
      },
      options2: {
        legend: "none",
        title: "Диапазоны CTR с точностью 95%",
        chartArea: {
          width: "50%",
        },
        isStacked: true,
        hAxis: {
          title: "CTR",
          minValue: 0,
        },
      },
      options3: {
        legend: "none",
        title: "Диапазоны CTR с точностью 99.7%",
        chartArea: {
          width: "50%",
        },
        isStacked: true,
        hAxis: {
          title: "CTR",
          minValue: 0,
        },
      },
    };
  },
  methods: {
    calc() {
      if (this.showing && this.clicks && this.faultVal && this.wishfulVal) {
        this.calcCtr();
        this.calcSelectionCtr();
        this.calcWilsonScoreIntervalForRealCTR();
        this.calcSelectionForCTR();
        this.calcWishfulCtr();
        this.checkRealAndWishCtrForCrossing();

        this.showTable = true;
      }
    },
    calcCtr() {
      this.realValue = this.clicks / this.showing;
    },
    calcSelectionCtr() {
      this.selectionArray = [];

      this.Sigma.forEach((value) => {
        this.selectionArray.push((value * value * this.realValue * (1 - this.realValue)) / (this.faultVal * this.faultVal));
      });
    },
    calcWilsonScoreIntervalForRealCTR() {
      this.ctrArray = [[], [], []];

      this.Sigma.forEach((value, index) => {
        let min =
          (this.realValue +
            (value * value) / (2 * this.showing) -
            value * Math.sqrt((this.realValue * (1 - this.realValue)) / this.showing + (value * value) / (4 * this.showing * this.showing))) /
          (1 + (value * value) / this.showing);

        let max =
          (this.realValue +
            (value * value) / (2 * this.showing) +
            value * Math.sqrt((this.realValue * (1 - this.realValue)) / this.showing + (value * value) / (4 * this.showing * this.showing))) /
          (1 + (value * value) / this.showing);
        console.log(min, max);

        // prepare array to draw schedule
        this.chartData[index][1] = [];
        this.chartData[index][1].push("Диапазон реального CTR на основе текущих данных");
        this.chartData[index][1].push(+(min * 100).toFixed(2));
        this.chartData[index][1].push(+((max - min) * 100).toFixed(2));
        this.chartData[index][1].push(`Диапазон реального CTR на основе текущих данных от ${+(min * 100).toFixed(2)}% до ${+(max * 100).toFixed(2)}%`);
        this.chartData[index][1].push("color: #cadd1b");

        this.ctrArray[index].push(min);
        this.ctrArray[index].push(max);
      });
    },
    calcSelectionForCTR() {
      this.showArray = [];
      this.needToShowArray = [[], [], []];
      this.ctrArray.forEach((value, index) => {
        let min = (this.Sigma[index] * this.Sigma[index] * value[0] * (1 - value[0])) / (this.faultVal * this.faultVal);
        let max = (this.Sigma[index] * this.Sigma[index] * value[1] * (1 - value[1])) / (this.faultVal * this.faultVal);
        max = max.toFixed() - this.showing;
        min = min.toFixed() - this.showing;

        if (max <= 0) {
          this.showArray.push(0);
        } else if (min < 0 && max > 0) {
          this.needToShowArray[index].push("");
          this.needToShowArray[index].push(`до ${max}`);
          this.showArray.push(1);
        } else {
          this.needToShowArray[index].push(`от ${min}`);
          this.needToShowArray[index].push(`до ${max}`);
          this.showArray.push(1);
        }
      });
    },
    calcWishfulCtr() {
      this.wishArray = [];

      this.wishfulCtrValueMin = this.wishfulVal - this.faultVal;
      this.wishfulCtrValueMax = this.wishfulVal + this.faultVal;
      // prepare array to draw schedule
      this.wishArray.push("Желаемый диапазон CTR");
      this.wishArray.push((this.wishfulCtrValueMin * 100).toFixed(2));
      this.wishArray.push(((this.wishfulCtrValueMax - this.wishfulCtrValueMin) * 100).toFixed(2));
      this.wishArray.push(`Желаемый диапазон CTR от ${(this.wishfulCtrValueMin * 100).toFixed(2)}% до ${(this.wishfulCtrValueMax * 100).toFixed(2)}%`);
      this.wishArray.push("color: #42c698");
    },
    checkRealAndWishCtrForCrossing() {
      this.checkArray = [];
      this.ctrArray.forEach((item) => {
        //check for crossing real and wish ctr
        if (
          ((this.wishfulCtrValueMin <= item[0] && this.wishfulCtrValueMax >= item[0]) ||
            (this.wishfulCtrValueMin <= item[1] && this.wishfulCtrValueMax >= item[1]) ||
            (item[0] <= this.wishfulCtrValueMin && item[1] >= this.wishfulCtrValueMax)) === true
        ) {
          this.checkArray.push(1);
        } else {
          this.checkArray.push(0);
        }
      });
    },
  },
  computed: {
    faultVal() {
      return this.fault / 100;
    },
    wishfulVal() {
      return this.wishfulCtr / 100;
    },
  },
};
</script>

<style>
body {
  background: #f7f7f7;
}

#btn {
  background-color: #665c84 !important;
  border-color: #665c84 !important;
}

td,
th {
  border: 1px solid #dee2e6 !important;
}
.text-big {
  font-size: 1.5rem;
}

#chart_div1 rect[fill="#3366cc"] {
  display: none !important;
}

#chart_div2 rect[fill="#3366cc"] {
  display: none !important;
}

#chart_div3 rect[fill="#3366cc"] {
  display: none !important;
}
#form {
  position: absolute;
  top: 55%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.moveInLeft {
  animation: moveInLeft 0.6s ease-in-out;
  transition: all 0.5s;
  transform: translate(-230%, -50%) !important;
}
.fadeIn {
  animation: fadeIn 0.6s ease-out;
}

@keyframes fadeIn {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}
</style>
