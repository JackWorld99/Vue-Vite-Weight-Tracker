<script setup>
import { ref, shallowRef, computed, watch, nextTick, onMounted } from "vue"
import Chart from "chart.js/auto"

const weights = ref([])
weights.value = JSON.parse(localStorage.getItem('weights')) || []

const weightChartEl = ref(null)

const weightChart = shallowRef(null)

const weightInput = ref(0)

var store = true

const currentWeight = computed(() => {
    return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})

const addWeight = () => {
    if (weightInput.value > 0) {
        weights.value.push({
            weight: weightInput.value,
            date: new Date().getTime(),
            time: new Date().toLocaleTimeString(),
        })
    }
}

const clearAll = () => {
    Swal.fire({
        title: 'Are you sure, clear all record?',
        text: "You won't be able to revert this!",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes, delete it!'
    }).then((result) => {
        if (result.isConfirmed) {

            localStorage.clear()
            store = false
            weights.value = [];

            Swal.fire({
                title: 'Deleted!',
                text: 'All url has been deleted.',
                icon: 'success',
                showConfirmButton: false,
                timer: 1000
            }).then(() => {
                location.reload();
            });
        }
    });
}

const delItem = (delID) => {
    weights.value.splice(delID, 1)
    localStorage.setItem("weights", JSON.stringify(weights))
}


watch(weights, (newWeights) => {
    if (store) localStorage.setItem('weights', JSON.stringify(newWeights))

    const weightArray = [...newWeights]
    const sortWeights = weightArray.sort((a, b) => a.date - b.date)

    if (weightChart.value) {
        weightChart.value.data.labels = sortWeights
            .map(weight => new Date(weight.date).toLocaleDateString())
            .slice(-7)

        weightChart.value.data.datasets[0].data = sortWeights
            .map(weight => weight.weight)
            .slice(-7)

        weightChart.value.update()
        weightInput.value = weights.value.map(weight => weight.weight)[0] || 0

        return
    }

    nextTick(() => {
        weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
            type: 'line',
            data: {
                labels: sortWeights.map(weight => new Date(weight.date).toLocaleDateString()),
                datasets: [{
                    label: "Weight",
                    data: sortWeights.map(weight => weight.weight),
                    backgroundColor: '#80e8ff',
                    borderColor: '#2681d6',
                    borderWidth: 1,
                    tension: 0.1,
                    fill: true,
                }]
            },
            options: {
                responsive: true,
            }
        })
    })

}, { deep: true })

if (weights.value.length != 0) {
    nextTick(() => {
        weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
            type: 'line',
            data: {
                labels: weights.value.sort((a, b) => a.date - b.date).map(weight => new Date(weight.date).toLocaleDateString()),
                datasets: [{
                    label: "Weight",
                    data: weights.value.sort((a, b) => a.date - b.date).map(weight => weight.weight),
                    backgroundColor: '#80e8ff',
                    borderColor: '#2681d6',
                    borderWidth: 1,
                    tension: 0.1,
                    fill: true,
                }]
            },
            options: {
                responsive: true,
            }
        })
    })
}

</script>

<template>
    <main>
        <h1>Weight Tracker</h1>
        <div class="current">
            <span>{{ currentWeight.weight }}</span>
            <small>Current weight (kg)</small>
        </div>

        <form @submit.prevent="addWeight">
            <input type="number" step="0.1" v-model="weightInput" />
            <input type="submit" value="Add weight" />
        </form>

        <div v-if="weights && weights.length > 0">
            <h2>Last 7 days</h2>
            <div class="canvas-box">
                <canvas ref="weightChartEl"></canvas>
            </div>
            <div class="weight-history">
                <div class="space">
                    <h2>Weight History</h2> <button @click="clearAll" type="button" style="margin-bottom: 1rem;">Clear
                        All</button>
                </div>
                <ul>
                    <li v-for="(weight, index) in weights">
                        <span>{{ weight.weight }} kg</span>
                        <div>
                            <small>
                                {{ new Date(weight.date).toLocaleDateString('en-GB') }} &ensp;{{ weight.time }}
                            </small> &emsp;
                            <i class="fa-solid fa-trash" @click="delItem(index)"></i>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </main>
</template>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'montserrat', sans-serif;
}

body {
    background-color: #6b6a69;
}

main {
    padding: 1.5rem;
}

h1 {
    font-size: 2em;
    text-align: center;
    margin-bottom: 2rem;
    color: rgb(255, 255, 255);
}

h2 {
    margin-bottom: 1rem;
    color: rgb(255, 255, 255);
    font-weight: 400;
}

.current {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 200px;
    height: 200px;

    text-align: center;
    background-color: white;
    border-radius: 999px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    border: 7px solid hwb(209 15% 16%);

    margin: 0 auto 2rem;
}

.current span {
    display: block;
    font-size: 2em;
    font-weight: bold;
    margin-bottom: 0.5rem;
}

.current small {
    color: #888;
    font-style: italic;
}

form {
    display: flex;
    margin-bottom: 2rem;
    border: 1px solid #AAA;
    border-radius: 0.5rem;
    overflow: hidden;
    transition: 200ms linear;
}

form:focus-within,
form:hover {
    border-color: #2681d6;
    border-width: 2px;
}

form input[type="number"] {
    appearance: none;
    outline: none;
    border: none;
    background-color: white;
    flex: 1 1 0%;
    padding: 1rem 1.5rem;
    font-size: 1.25rem;
}

form input[type="submit"] {
    appearance: none;
    outline: none;
    border: none;
    cursor: pointer;
    background-color: #2681d6;
    padding: 0.5rem 1rem;
    color: white;
    font-size: 1.25rem;
    font-weight: 700;
    transition: 200ms linear;
    border-left: 3px solid transparent;
}

form input[type="submit"]:hover {
    background-color: #0aa6fa;
    color: white;
    border-left-color: #2681d6;
}

.canvas-box {
    width: 100%;
    max-width: 100%;
    background-color: white;
    padding: 1rem;
    border-radius: 0.5rem;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    margin-bottom: 2rem;
}

.weight-history ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.weight-history ul li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 1rem;
    cursor: pointer;
    border-radius: 0.5rem;
}

.space {
    display: flex;
    justify-content: space-between;
}

.weight-history ul li:nth-child(even) {
    background-color: #2681d6;
}

.weight-history ul li:nth-child(odd) {
    background-color: #0aa6fa;
}

.weight-history ul li:hover {
    background-color: #1e49f7;
}

.weight-history ul li:last-of-type {
    border-bottom: none;
}

.weight-history ul li span {
    display: block;
    font-size: 1.25rem;
    font-weight: 500;
    margin-right: 1rem;
    color: white;
}

.weight-history ul li small {
    color: white;
    font-style: italic;
}

.fa-solid,
.fa-trash {
    color: white;
}

.space button[type="button"] {
    appearance: none;
    outline: none;
    border: none;
    cursor: pointer;
    background-color: #2681d6;
    color: white;
    border-radius: 0.4rem;
    padding: 0.5rem 0.8rem;
}

.space button[type="button"]:hover {
    background-color: #0aa6fa;
    color: white;
    border-left-color: #2681d6;
}
</style>
