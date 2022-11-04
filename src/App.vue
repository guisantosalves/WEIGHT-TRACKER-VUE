<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

// ref -> is like useState

const weights = ref([]);

const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(60.6);

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

// when the user click in the submit button will active this func
const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
};

watch(weights, newWeights => {
  const ws = [...newWeights] // -> unwrap from weights here
  
  if(weightChart.value){

    // it will execut only if we already have a value inside the weightChart
    weightChart.value.data.labels = ws
        .sort((a, b)=> a.date - b.date)
        .map(w => new Date(w.date).toLocaleDateString())
        .slice(-7)
    
    weightChart.value.data.datasets[0].data = ws
      .sort((a, b)=> a.date - b.date)
      .map(w => w.weight)
      .slice(-7)

      weightChart.value.update()

      return
  }

  // nextTick allows you to execute code after you have changed 
  //some data and Vue.js has updated the virtual DOM based on your data change, but before 
  //the browser has rendered that change on the page.

  // labels -> eixo x / datasets -> eixo y
  nextTick(()=>{
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type: 'line',
      data: {
        labels: ws
        .sort((a, b)=> a.date - b.date)
        .map(w => new Date(w.date).toLocaleDateString()),
        datasets: [
          {
            label: 'Weight',
            data: ws
                  .sort((a, b)=> a.date - b.date)
                  .map(w => w.weight),
            backgroundColor: 'rgba(255, 105, 180, 0.2)',
            borderColor: 'rgb(255, 105, 180)',
            borderWidth: 1,
            fill: true
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false
      }
    })
  })

  
}, {deep: true})

</script>

<template>
  <main>
    <h1>Weight Tracker</h1>

    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current Weight (KG)</small>
    </div>

    <form @submit.prevent="addWeight">
      <div class="containerForm">
        <input type="number" step="0.1" v-model="weightInput" />
      </div>
      <input type="submit" value="Add weight" />
    </form>

    <div v-if="weights && weights.length > 0">
      <h2>Last 7 days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <div class="weight-history">
        <h2>weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}KG</span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>

    </div>

  </main>
</template>

<style scoped></style>
