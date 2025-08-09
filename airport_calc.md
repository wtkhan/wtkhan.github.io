---
layout: default
title: Airport Departure Calculator
tagline: When should you leave for the aiport?
permalink: /airport_calc.html
description: " "
ref: now
order: 2
---

{% raw %}

<form id="calcForm" autocomplete="off">
  <div class="form-group">
    <label for="flightDateTime">Flight date & time:</label>
    <br>
    <input type="datetime-local" id="flightDateTime" required>
  </div>
<br>
<br>
  <div class="form-group">
    <label for="baseline">Baseline minutes before departure (default is 45):</label>
    <input type="number" id="baseline" min="1" value="45" required>
  </div>
<br>
<br>

  <div class="form-group">
    <label for="commute">In minutes, what is your estimated commute to the airport? (default 20)</label>
    <br>
    <input type="number" id="commute" min="0" value="20" required>
  </div>
<br>
<br>

  <div class="form-group">
    <label>How are you getting to the airport? (adds time)</label>
    <br>
    <div class="radio-group">
      <label><input type="radio" name="transport" value="rideshare">Rideshare / Public Transit (+15 min)</label>
      <label><input type="radio" name="transport" value="car" checked>Personal Car (+5 min)</label>
    </div>
  </div>
<br>
<br>

  <div class="form-group">
    <label>Are you heading to a busy airport (JFK, ORD, ATL)? (adds time)</label>
    <br>
    <div class="radio-group">
      <label><input type="radio" name="busyAirport" value="yes">Yes (+20 min)</label>
      <label><input type="radio" name="busyAirport" value="no" checked>No (+10 min)</label>
    </div>
  </div>
<br>
<br>

  <div class="form-group">
    <label>Do you have slow movers (children, seniors) in your group? (adds time)</label>
    <br>
    <div class="radio-group">
      <label><input type="radio" name="slowMovers" value="yes">Yes (+10 min)</label>
      <label><input type="radio" name="slowMovers" value="no" checked>No (0 min)</label>
    </div>
  </div>
<br>
<br>

  <div class="form-group">
    <label>Do you have expedited security (TSA PreCheck, CLEAR)? (subtracts time)</label>
    <br>
    <div class="radio-group">
      <label><input type="radio" name="expedited" value="yes">Yes (-5 min)</label>
      <label><input type="radio" name="expedited" value="no" checked>No (0 min)</label>
    </div>
  </div>
<br>
<br>

  <div class="form-group">
    <label>Do you need to print tickets or check bags at the counter? (adds time)</label>
    <br>
    <div class="radio-group">
      <label><input type="radio" name="checkin" value="yes">Yes (+20 min)</label>
      <label><input type="radio" name="checkin" value="no" checked>No (0 min)</label>
    </div>
  </div>
<br>
<br>

  <div class="form-group">
    <label>Do you want to get coffee or food at the airport? (adds time)</label>
    <br>
    <div class="radio-group">
      <label><input type="radio" name="coffee" value="yes">Yes (+20 min)</label>
      <label><input type="radio" name="coffee" value="no" checked>No (0 min)</label>
    </div>
  </div>
<br>
<br>

  <button type="submit" id="calcBtn">Calculate</button>
  <button type="reset" id="resetBtn">Reset</button>
</form>

<div id="result"></div>

<script>
// ---------- Configuration ---------- //
const CONFIG = {
  baselineDefault: 45,
  commuteDefault: 20,
  transportAdd: { rideshare: 15, car: 5 },
  busyAdd: { yes: 20, no: 10 },
  slowAdd: { yes: 10, no: 0 },
  expeditedSub: { yes: 5, no: 0 },
  checkinAdd: { yes: 20, no: 0 },
  coffeeAdd: { yes: 20, no: 0 }
};

// ---------- Helper Functions ---------- //
function formatLeaveTime(date) {
  const dateOpts = { year: 'numeric', month: 'short', day: 'numeric' };
  const timeOpts = { hour: 'numeric', minute: '2-digit', hour12: true };
  return `${date.toLocaleDateString(undefined, dateOpts)}, ${date.toLocaleTimeString(undefined, timeOpts)}`;
}

function calculateLeaveTime(flightDateTime, data) {
  const transportAdd = CONFIG.transportAdd[data.transport];
  const busyAdd = CONFIG.busyAdd[data.busyAirport];
  const slowAdd = CONFIG.slowAdd[data.slowMovers];
  const expeditedSub = CONFIG.expeditedSub[data.expedited];
  const checkinAdd = CONFIG.checkinAdd[data.checkin];
  const coffeeAdd = CONFIG.coffeeAdd[data.coffee];

  const totalSubtract = data.baseline + data.commute + transportAdd + busyAdd + slowAdd + checkinAdd + coffeeAdd - expeditedSub;
  return new Date(flightDateTime.getTime() - totalSubtract * 60000);
}

// ---------- Form Handling ---------- //
function gatherFormValues() {
  const f = document.getElementById('calcForm');
  const flightDateTimeStr = f.flightDateTime.value;
  if (!flightDateTimeStr) throw new Error('Flight date & time is required');

  // Validate format and components
  const timeRegex = /^\d{4}-\d{2}-\d{2}T(\d{2}):(\d{2})$/;
  const m = flightDateTimeStr.match(timeRegex);
  if (!m) throw new Error('Invalid flight date & time format');
  const hour = parseInt(m[1], 10);
  const minute = parseInt(m[2], 10);
  if (hour < 0 || hour > 23 || minute < 0 || minute > 59) throw new Error('Invalid time value');

  const flightDateTime = new Date(flightDateTimeStr);
  if (isNaN(flightDateTime.getTime())) throw new Error('Invalid flight date & time');

  const baseline = parseInt(f.baseline.value, 10) || CONFIG.baselineDefault;
  const commuteRaw = parseInt(f.commute.value, 10);
  const commute = isNaN(commuteRaw) ? CONFIG.commuteDefault : commuteRaw;

  return {
    flightDateTime,
    baseline,
    commute,
    transport: f.transport.value,
    busyAirport: f.busyAirport.value,
    slowMovers: f.slowMovers.value,
    expedited: f.expedited.value,
    checkin: f.checkin.value,
    coffee: f.coffee.value
  };
}

function updateResult() {
  try {
    const data = gatherFormValues();
    const leaveDate = calculateLeaveTime(data.flightDateTime, data);
    const formatted = formatLeaveTime(leaveDate);
    document.getElementById('result').textContent = `You should leave at ${formatted}`;
    document.getElementById('result').className = '';
  } catch (e) {
    document.getElementById('result').textContent = e.message;
    document.getElementById('result').className = 'error';
  }
}

document.getElementById('calcForm').addEventListener('submit', e => {
  e.preventDefault();
  updateResult();
});

document.addEventListener('keydown', e => {
  if (e.key === 'Enter' && document.activeElement.closest('form')) {
    e.preventDefault();
    updateResult();
  }
});

// ---------- Reset Handling ---------- //
function setDefaults() {
  const today = new Date().toISOString().split('T')[0];
  document.getElementById('flightDateTime').value = `${today}T12:00`;
  document.getElementById('baseline').value = CONFIG.baselineDefault;
  document.getElementById('commute').value = CONFIG.commuteDefault;
}
document.getElementById('resetBtn').addEventListener('click', () => {
  setDefaults();
  document.getElementById('result').textContent = '';
});

// ---------- Initialize ---------- //
setDefaults();

// ---------- Simple Unit Tests (Optional) ---------- //
function runTests() {
  const tests = [
    {
      name: 'Baseline example (08/09/2025 10:30 AM)',
      input: {
        flightDateTime: new Date('2025-08-09T10:30'),
        baseline: 45,
        commute: 20,
        transport: 'car',
        busyAirport: 'no',
        slowMovers: 'no',
        expedited: 'no',
        checkin: 'no',
        coffee: 'no'
      },
      expected: 'Aug 9, 2025, 9:10 AM'
    },
    {
      name: 'All overrides (08/09/2025 10:30 AM)',
      input: {
        flightDateTime: new Date('2025-08-09T10:30'),
        baseline: 45,
        commute: 0,
        transport: 'rideshare',
        busyAirport: 'yes',
        slowMovers: 'yes',
        expedited: 'yes',
        checkin: 'yes',
        coffee: 'yes'
      },
      expected: 'Aug 9, 2025, 8:25 AM'
    }
  ];
  let passed = 0;
  tests.forEach(t => {
    const leave = calculateLeaveTime(t.input.flightDateTime, t.input);
    const formatted = formatLeaveTime(leave);
    if (formatted === t.expected) {
      console.log(`✓ ${t.name}`);
      passed++;
    } else {
      console.error(`✗ ${t.name}\n  expected: ${t.expected}\n  got: ${formatted}`);
    }
  });
  console.log(`Test summary: ${passed}/${tests.length} passed.`);
}
window.addEventListener('load', runTests);
</script>
<footer style="margin-top:1.5rem; font-size:.9rem; color:#555;">
  The calculator assumes that the user is flying from a US airport. It is inspired by Nate Silver's
  <a href="https://www.natesilver.net/p/how-soon-should-you-arrive-at-the-airport" target="_blank">
    post</a> and vibe-coded with <a href="https://ollama.com/library/gpt-oss" target="_blank"> GPT-OSS</a>.
</footer>

{% endraw %}