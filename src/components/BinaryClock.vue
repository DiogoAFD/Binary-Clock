<template>
  <div class="binary-clock-layout">
    <div class="controls">
      <label><input type="checkbox" v-model="showBitValues" /> Tvíundar gildi</label>
      <label><input type="checkbox" v-model="showLabels" /> Sýna merki</label>
      <label><input type="checkbox" v-model="showCalculations" /> Sýna útreikninga</label>
      <button @click="toggleTimer" class="timer-toggle-btn">{{ timerRunning ? 'Stoppa klukkuna' : 'Halda áfram' }}</button>
    </div>
    <div class="binary-clock">
      <div class="clock-label">Klukka</div>
      <div class="clock-section">
        <div v-if="showBitValues" class="extra-bits-row-with-label">
          <span v-if="showLabels" class="extra-bits-label">Tvíundar gildi:</span>
          <div class="bit-circles-row extra-bits-row">
            <span v-for="bit in bitValues" :key="bit" class="bit-circle filled bit-value-circle">
              <span class="bit-value-text">{{ bit }}</span>
            </span>
          </div>
        </div>
        <div v-for="(part, i) in timeParts" :key="part.label" class="clock-row">
          <div v-if="showLabels" class="row-label">
            {{ part.label }}: {{ part.value }} ({{ part.binary }})
          </div>
          <div class="binary-row-horizontal">
            <div class="bit-circles-row">
              <span v-for="(bit, j) in part.binary" :key="j" class="bit-circle" :class="{ filled: bit === '1' }"></span>
            </div>
            <div v-if="showCalculations" class="calculation">
              {{ calculationString(part.binary, bitValues) }}
            </div>
          </div>
        </div>
      </div>
      <hr class="section-separator" />
      <div class="date-label">Dagsetning</div>
      <div class="date-section-layout">
        <div class="date-inputs">
          <label>
            Dagur:
            <input type="text" v-model="inputDay" maxlength="2" @keydown="numberOnly" />
          </label>
          <label>
            Mánuður:
            <input type="text" v-model="inputMonth" maxlength="2" @keydown="numberOnly" />
          </label>
          <label>
            Ár:
            <input type="text" v-model="inputYear" maxlength="4" @keydown="numberOnly" />
          </label>
        </div>
        <div class="date-section">
          <div v-if="showBitValues" class="extra-bits-row-with-label">
            <span v-if="showLabels" class="extra-bits-label">Tvíundar gildi:</span>
            <div class="bit-circles-row extra-bits-row">
              <span v-for="bit in dateBitValues" :key="bit" class="bit-circle filled bit-value-circle date">
                <span class="bit-value-text">{{ bit }}</span>
              </span>
            </div>
          </div>
          <div v-for="(part, i) in dateParts" :key="part.label" class="date-row">
            <div v-if="showLabels" class="row-label">
              {{ part.label }}: {{ part.value }} ({{ part.binary }})
            </div>
            <div class="binary-row-horizontal">
              <div class="bit-circles-row">
                <span v-for="(bit, j) in part.binary" :key="j" class="bit-circle date" :class="{ filled: bit === '1' }"></span>
              </div>
              <div v-if="showCalculations" class="calculation">
                {{ calculationString(part.binary, dateBitValues) }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div> <!-- This closes binary-clock-layout -->
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted, onUnmounted } from 'vue';

export default defineComponent({
 name: 'BinaryClock',
 setup() {
 const showBitValues = ref(false);
 const showLabels = ref(false);
 const showCalculations = ref(false);
 const now = ref(new Date());
 let timer: number | null = null;
 const timerRunning = ref(true);

 // Date input fields
 const inputDay = ref('');
 const inputMonth = ref('');
 const inputYear = ref('');

 onMounted(() => {
 // Set default date inputs to today
 const today = new Date();
 inputDay.value = today.getDate().toString().padStart(2, '0');
 inputMonth.value = (today.getMonth() +1).toString().padStart(2, '0');
 inputYear.value = today.getFullYear().toString();
 startTimer();
 });
 onUnmounted(() => {
 stopTimer();
 });

 function startTimer() {
 if (!timerRunning.value) timerRunning.value = true;
 if (timer === null) {
 timer = window.setInterval(() => {
 now.value = new Date();
 },1000);
 }
 }
 function stopTimer() {
 if (timer !== null) {
 clearInterval(timer);
 timer = null;
 timerRunning.value = false;
 }
 }
 function toggleTimer() {
 if (timerRunning.value) {
 stopTimer();
 } else {
 startTimer();
 }
 }

 const bitValues = [32,16,8,4,2,1];
 const dateBitValues = [128,64,32,16,8,4,2,1];

 const timeParts = computed(() => {
 const labels = ['Klst', 'Mín', 'Sek'];
 const values = [now.value.getHours(), now.value.getMinutes(), now.value.getSeconds()];
 return values.map((v, i) => ({
 label: labels[i],
 value: v,
 binary: v.toString(2).padStart(6, '0')
 }));
 });

 const dateParts = computed(() => {
 // Use input values for date
 const yearText = inputYear.value.padStart(4, '0');
 const monthText = inputMonth.value.padStart(2, '0');
 const dayText = inputDay.value.padStart(2, '0');

 const yearFirstTwo = parseInt(yearText.substring(0,2),10) ||0;
 const yearLastTwo = parseInt(yearText.substring(2,4),10) ||0;
 const month = parseInt(monthText,10) ||0;
 const day = parseInt(dayText,10) ||0;
 const labels = ['Dagur', 'Mánuður', 'Ár (1-2)', 'ÁR (3-4)'];
 const values = [day, month, yearFirstTwo, yearLastTwo];
 return values.map((v, i) => ({
 label: labels[i],
 value: v,
 binary: v.toString(2).padStart(8, '0')
 }));
 });

 function calculationString(binary: string, bits: number[]) {
 return binary.split('').map((b, i) => b === '1' ? bits[i] :0).join('+');
 }

 // Only allow numbers, backspace, and delete in date inputs
 function numberOnly(e: KeyboardEvent) {
 const allowed = ["Backspace", "Delete", "ArrowLeft", "ArrowRight", "Tab"];
 if (!/\d/.test(e.key) && !allowed.includes(e.key)) {
 e.preventDefault();
 }
 }

 return {
 showBitValues,
 showLabels,
 showCalculations,
 timeParts,
 dateParts,
 bitValues,
 dateBitValues,
 calculationString,
 inputDay,
 inputMonth,
 inputYear,
 numberOnly,
 timerRunning,
 toggleTimer
 };
 }
});
</script>

<style scoped>
.binary-clock-layout {
 display: flex;
 flex-direction: row;
 align-items: flex-start;
}
.controls {
 display: flex;
 flex-direction: column;
 margin-right:2em;
 margin-top:2em;
}
.controls label {
 margin-bottom:1em;
 white-space: nowrap;
}
.timer-toggle-btn {
 margin-top:1em;
 padding:0.5em1.2em;
 font-size:1em;
 font-weight:bold;
 background:#2c3e50;
 color:#fff;
 border:none;
 border-radius:6px;
 cursor:pointer;
}
.timer-toggle-btn:hover {
 background:#34495e;
}
.binary-clock {
 font-family: Arial, sans-serif;
 padding:2em;
 flex:1;
}
.clock-label {
 font-size:1.3em;
 font-weight: bold;
 color: #2c3e50;
 margin-bottom:1.5em;
 text-align: left;
}
.clock-section, .date-section {
 margin-bottom:2em;
}
.date-section-layout {
 display: flex;
 flex-direction: row;
 align-items: flex-start;
}
.date-inputs {
 display: flex;
 flex-direction: column;
 margin-right:2em;
 margin-top:0.5em;
}
.date-inputs label {
 margin-bottom:1em;
 font-weight: bold;
}
.date-inputs input[type="text"] {
 width:60px;
 padding:0.3em;
 font-size:1em;
 text-align: right;
}
.section-separator {
 border: none;
 border-top:2px solid #bbb;
 margin-top:2em;
 margin-bottom:4em;
}
.date-label {
 font-size:1.3em;
 font-weight: bold;
 color: #2c3e50;
 margin-bottom:1.5em;
 text-align: left;
}
.clock-row, .date-row {
 display: flex;
 align-items: center;
 margin-bottom:1em;
}
.row-label {
 min-width:180px;
 font-weight: bold;
}
.binary-row-horizontal {
 display: flex;
 align-items: center;
}
.bit-labels-row {
 display: flex;
 margin-bottom: 0.5em;
}
.extra-bits-row-with-label {
 display: flex;
 align-items: center;
 margin-bottom: 0.5em;
}
.extra-bits-label {
 min-width:180px;
 font-weight: bold;
 font-size:1em;
 color: darkred;
 white-space: nowrap;
 display: inline-block;
}
.extra-bits-row {
 display: flex;
}
.bit-value-circle {
 position: relative;
 display: flex;
 align-items: center;
 justify-content: center;
}
.bit-value-text {
 position: absolute;
 left:50%;
 top:50%;
 transform: translate(-50%, -50%);
 color: #fff;
 font-size:1em;
 font-weight: bold;
 pointer-events: none;
}
.bit-label {
 width:30px;
 text-align: center;
 color: darkred;
 font-size:1em;
 font-weight: bold;
}
.bit-circles-row {
 display: flex;
}
.bit-circle {
 width:30px;
 height:30px;
 border-radius:50%;
 border:2px solid gray;
 margin-right:8px;
 background: white;
 display: inline-block;
 position: relative;
}
.bit-circle.filled {
 background: darkblue;
 border-color: darkblue;
}
.bit-circle.date.filled,
.bit-value-circle.date {
 background: darkgreen;
 border-color: darkgreen;
}
.calculation {
 margin-left:1em;
 color: darkblue;
 font-size:1em;
 font-weight: bold;
}
</style>
