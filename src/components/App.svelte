<script>
  import * as d3 from "d3";
  import { line, scaleLinear, axisBottom, axisLeft } from 'd3';
  let score;
  try {
    let savedScore = localStorage.getItem("score") || false;
    score = savedScore ? JSON.parse(savedScore) : {
      stay: {win: 0, loss: 0},
      switch: {win: 0, loss: 0},
    }
  } catch (e) {
    score = {
      stay: {win: 0, loss: 0},
      switch: {win: 0, loss: 0},
    }
  }
  let path = ""; // "stay" | "switch"
  let gamePhase = 0;
  let userSelection = null;
  let hostRevealed = null;
  let gameResult = "";
  function selectPrizeDoor() {
    return Math.floor(Math.random() * 3);
  }
  let prizeDoor = selectPrizeDoor();

  function initialize() {
    gamePhase = 0;
    gameResult = "";
    prizeDoor = selectPrizeDoor();
    userSelection = null;

  }
  initialize();
  function openDoor(doorNumber) {
    if ((prizeDoor === doorNumber)||(prizeDoor === doorNumber-3)||(prizeDoor === doorNumber-6)) {
      let new_elt = document.createElement('img');
      new_elt.src = "car.svg";
      new_elt.className = "door";
      document.getElementsByClassName("door")[doorNumber].replaceWith(new_elt);
    } else {
      let new_elt = document.createElement('img');
      new_elt.src = "goat.svg";
      new_elt.className = "door";
      document.getElementsByClassName("door")[doorNumber].replaceWith(new_elt);
    }
  }
  function closeDoor(doorNumber) {
    let new_elt = document.createElement('img');
    new_elt.src = "closed-door.svg";
    new_elt.className = "door";
    document.getElementsByClassName("door")[doorNumber].replaceWith(new_elt);
  }
  function selectDoor(doorNumber) {
    if (gamePhase === 0) {
      userSelection = doorNumber;
    }
  } 
  function autoselectStay(iteration) {
    userSelection = Math.floor(Math.random() * 3);
    setTimeout(() => {  revealHost(); }, 100);
    setTimeout(() => {  userStay(); }, 100);
    setTimeout(() => {  revealAll(); }, 200);
    setTimeout(() => {  data.push({ trial: iteration+1, percentage: Math.round((score.stay.win / (score.stay.win + score.stay.loss)) * 100) });  }, 200);
    setTimeout(() => {  drawChart(); }, 300);
    setTimeout(() => {  restartGame(); }, 300);
  }
  function autoselectSwitch(iteration) {
    userSelection = Math.floor(Math.random() * 3);
    setTimeout(() => {  revealHost(); }, 100);
    setTimeout(() => {  userSwitch(); }, 150);
    setTimeout(() => {  revealAll(); }, 200);
    setTimeout(() => {  dataSwitch.push({ trial: iteration+1, percentage: Math.round((score.switch.win / (score.switch.win + score.switch.loss)) * 100) });  }, 200);
    setTimeout(() => {  drawLineChart(); }, 300);
    setTimeout(() => {  restartGame(); }, 300);
  }
  function restartGame() {
    closeDoor(0);
    closeDoor(1);
    closeDoor(2);
    closeDoor(3);
    closeDoor(4);
    closeDoor(5);
    closeDoor(6);
    closeDoor(7);
    closeDoor(8);
    initialize();
  }
  function next() {
    if (userSelection !== null) {
      gamePhase++;
    }
  }
  function revealHost() {
    if (userSelection !== null) {
      if (userSelection === prizeDoor) {
        const doors = [0, 1, 2];
        doors.splice(userSelection, 1);
        let randomBinary = new Date();
        randomBinary = randomBinary.getMilliseconds() % 2;
        hostRevealed = doors[randomBinary];
      } 
      else {
        hostRevealed = 3 - userSelection - prizeDoor;
      }
      openDoor(hostRevealed);
      openDoor(hostRevealed+3);
      openDoor(hostRevealed+6);
    }
    gamePhase++;
  } 
  function userStay() {
    path = "stay";
    gamePhase++;
  } 
  function userSwitch() {
    userSelection = 3 - userSelection - hostRevealed;
    path = "switch";
    gamePhase++;
  } 
  function revealAll() {
    openDoor(0);
    openDoor(1);
    openDoor(2);
    openDoor(3);
    openDoor(4);
    openDoor(5);
    openDoor(6);
    openDoor(7);
    openDoor(8);
    if (userSelection === prizeDoor) {
      gameResult = "You won!  But did you optimize your odds or were you just lucky? In other words, is it better to stay or to switch?"
    } else {
      gameResult = "You lost! Is there a way to optimize your chance for next time? In other words, is it better to stay or to switch?"
    }
    score[path][["loss", "win"][Number(userSelection === prizeDoor)]]++;
    localStorage.setItem("score", JSON.stringify(score));
    updatePieChartStay();
    updatePieChartSwitch();
    // updateLineChart();
    gamePhase++;
  }
  function updatePieChartStay() {
    d3.select('#pieChartStay').selectAll('svg').remove();
    const data = [
        { label: 'Win', value: score.stay.win },
        { label: 'Loss', value: score.stay.loss }
    ];
    const width = 300;
    const height = 300;
    const radius = Math.min(width, height) / 2;

    const svg = d3.select('#pieChartStay')
        .append('svg')
        .attr('width', width)
        .attr('height', height)
        .append('g')
        .attr('transform', `translate(${width / 2},${height / 2})`)
        .attr("stroke", "rgba(2, 35, 64, .25)")
        .style("stroke-width", "1px");

    const color = d3.scaleOrdinal()
        .domain(data.map(d => d.label))
        .range(['#96FF98', '#FF595C']);

    const pie = d3.pie()
        .sort(null)
        .value(d => d.value);

    const path = d3.arc()
        .outerRadius(radius - 10)
        .innerRadius(0);

    const label = d3.arc()
        .outerRadius(radius - 40)
        .innerRadius(radius - 40);

    const arc = svg.selectAll('.arc')
        .data(pie(data))
        .enter().append('g')
        .attr('class', 'arc');

    arc.append('path')
        .attr('d', path)
        .attr('fill', d => color(d.data.label));

    arc.append('text')
        .attr('transform', d => `translate(${label.centroid(d)})`)
        .attr('dy', '0.35em')
        .text(d => Math.round(d.data.value/(score.stay.win + score.stay.loss) * 100)+"%")
        .style("text-anchor", "middle")
        .style("font-size", 17)
        .style('font-family', 'Barlow');
}
function updatePieChartSwitch() {
    d3.select('#pieChartSwitch').selectAll('svg').remove();
    const data = [
        { label: 'Win', value: score.switch.win },
        { label: 'Loss', value: score.switch.loss }
    ];
    const width = 300;
    const height = 300;
    const radius = Math.min(width, height) / 2;

    const svg = d3.select('#pieChartSwitch')
        .append('svg')
        .attr('width', width)
        .attr('height', height)
        .append('g')
        .attr('transform', `translate(${width / 2},${height / 2})`)
        .attr("stroke", "rgba(2, 35, 64, .25)")
        .style("stroke-width", "1px");

    const color = d3.scaleOrdinal()
        .domain(data.map(d => d.label))
        .range(['#96FF98', '#FF595C']);

    const pie = d3.pie()
        .sort(null)
        .value(d => d.value);

    const path = d3.arc()
        .outerRadius(radius - 10)
        .innerRadius(0);

    const label = d3.arc()
        .outerRadius(radius - 40)
        .innerRadius(radius - 40);

    const arc = svg.selectAll('.arc')
        .data(pie(data))
        .enter().append('g')
        .attr('class', 'arc');

    arc.append('path')
        .attr('d', path)
        .attr('fill', d => color(d.data.label));

    arc.append('text')
        .attr('transform', d => `translate(${label.centroid(d)})`)
        .attr('dy', '0.35em')
        .text(d => Math.round(d.data.value/(score.switch.win + score.switch.loss) * 100)+"%")
        .style("text-anchor", "middle")
        .style("font-size", 17)
        .style('font-family', 'Barlow');
}
let data = [
    ];
let dataSwitch = [
    ];
function initializeLineChart(){
  d3.select('#lineChart').selectAll('svg').remove();
      const margin = { top: 20, right: 20, bottom: 30, left: 40 };
      const width = 400;
      const height = 200;

      const svg = d3.select('#lineChart')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom+6)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = scaleLinear()
        .domain([1, 100])
        .range([0, width]);

      const y = scaleLinear()
        .domain([0, 100])
        .range([height, 0]);

      const xAxis = axisBottom(x);
      const yAxis = axisLeft(y);

      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(xAxis)
        .attr("font-family","Barlow");

      svg.append('g')
        .call(yAxis)
        .attr("font-family","Barlow");


      svg.append("text")
        .attr("class", "x label")
        .attr("text-anchor", "middle")
        .attr("x", width/2)
        .attr("y", height +35)
        .text("NUMBER OF TRIALS");
      svg.append("text")
        .attr("class", "y label")
        .attr("text-anchor", "middle")
        .attr("y", -27)
        .attr("x", -100)
        // .attr("dy", ".75em")
        .attr("transform", "rotate(-90)")
        .text("SUCCESS RATE");
}
function initializeLineChartSwitch(){
  d3.select('#lineChartSwitch').selectAll('svg').remove();
      const margin = { top: 20, right: 20, bottom: 30, left: 40 };
      const width = 400;
      const height = 200;

      const svg = d3.select('#lineChartSwitch')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom+6)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = scaleLinear()
        .domain([1, 100])
        .range([0, width]);

      const y = scaleLinear()
        .domain([0, 100])
        .range([height, 0]);

      const xAxis = axisBottom(x);
      const yAxis = axisLeft(y);

      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(xAxis)
        .attr("font-family","Barlow");

      svg.append('g')
        .call(yAxis)
        .attr("font-family","Barlow");


      svg.append("text")
        .attr("class", "x label")
        .attr("text-anchor", "middle")
        .attr("x", width/2)
        .attr("y", height +35)
        .text("NUMBER OF TRIALS");
      svg.append("text")
        .attr("class", "y label")
        .attr("text-anchor", "middle")
        .attr("y", -27)
        .attr("x", -100)
        // .attr("dy", ".75em")
        .attr("transform", "rotate(-90)")
        .text("SUCCESS RATE");
}
function drawChart() {
  d3.select('#lineChartSwitch').selectAll('svg').remove();
      const margin = { top: 20, right: 20, bottom: 30, left: 40 };
      const width = 400;
      const height = 200;

      const svg = d3.select('#lineChartSwitch')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom+6)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = scaleLinear()
        .domain([1, 100])
        .range([0, width]);

      const y = scaleLinear()
        .domain([0, 100])
        .range([height, 0]);

      const xAxis = axisBottom(x);
      const yAxis = axisLeft(y);

      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(xAxis)
        .attr("font-family","Barlow");

      svg.append('g')
        .call(yAxis)
        .attr("font-family","Barlow");


      svg.append("text")
        .attr("class", "x label")
        .attr("text-anchor", "middle")
        .attr("x", width/2)
        .attr("y", height +35)
        .text("NUMBER OF TRIALS");
      svg.append("text")
        .attr("class", "y label")
        .attr("text-anchor", "middle")
        .attr("y", -27)
        .attr("x", -100)
        // .attr("dy", ".75em")
        .attr("transform", "rotate(-90)")
        .text("SUCCESS RATE");

      const lineGenerator = line()
        .x(d => x(d.trial))
        .y(d => y(d.percentage));

      const path = svg.append('path')
        .datum(data)
        .attr('fill', 'none')
        .attr('stroke', 'steelblue')
        .attr('stroke-width', 1.5)
        .attr('d', lineGenerator);

  }
  function drawLineChart() {
    d3.select('#lineChartSwitch').selectAll('svg').remove();
      const margin = { top: 20, right: 20, bottom: 30, left: 40 };
      const width = 400;
      const height = 200;

      const svg = d3.select('#lineChartSwitch')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom+6)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      const x = scaleLinear()
        .domain([1, 100])
        .range([0, width]);

      const y = scaleLinear()
        .domain([0, 100])
        .range([height, 0]);

      const xAxis = axisBottom(x);
      const yAxis = axisLeft(y);

      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(xAxis);

      svg.append('g')
        .call(yAxis);

      const lineGenerator = line()
        .x(d => x(d.trial))
        .y(d => y(d.percentage));

      const path = svg.append('path')
        .datum(dataSwitch)
        .attr('fill', 'none')
        .attr('stroke', 'steelblue')
        .attr('stroke-width', 1.5)
        .attr('d', lineGenerator);
      svg.append("text")
        .attr("class", "x label")
        .attr("text-anchor", "middle")
        .attr("x", width/2)
        .attr("y", height +35)
        .text("NUMBER OF TRIALS");
      svg.append("text")
      .attr("class", "y label")
      .attr("text-anchor", "middle")
      .attr("y", -27)
      .attr("x", -100)
      // .attr("dy", ".75em")
      .attr("transform", "rotate(-90)")
      .text("SUCCESS RATE");

      // data.forEach((d, i) => {
      //   setTimeout(() => {
      //     path.transition()
      //       .attr('d', lineGenerator(data.slice(0, i + 1)));
      //   }, i * 1000); // Adjust the delay time as needed
      // });
  }
</script>

<main>
  <div class = "title-page">
    <img src = "title-page.svg"/>
    <button class = "aboutBtn"><img src = "question-sign.png"/></button>
    <button class = "btn" on:click={() => {initializeLineChart(); initializeLineChartSwitch();}}>Let's do this!</button>
  </div>
  <div class = "game">
    <p>In front of you are three doors. Behind one door is a car 🚗 and behind the other two are goats 🐐🐐.</p>
    <div class="row">
      <div class = "column" on:click={() => selectDoor(0)}
        style={`border: solid 5px ${userSelection === 0 ? "black" : "#fef2cf"}`}
        >
        <img class = "door" src= "closed-door.svg"/>
      </div>
      <div class = "column" on:click={() => selectDoor(1)}
        style={`border: solid 5px ${userSelection === 1 ? "black" : "#fef2cf"}`}
        >
        <img class = "door"  src= "closed-door.svg"/>
      </div>
      <div class = "column" on:click={() => selectDoor(2)}
        style={`border: solid 5px ${userSelection === 2 ? "black" : "#fef2cf"}`}
        >
        <img class = "door" src= "closed-door.svg"/>
      </div>
    </div>
  </div>
  {#if gamePhase === 0}
    <div class="phase0">
      <p>Select a door.</p>
      <button class = "btn" on:click={next}>Next</button>
      <br>
    </div>
  {/if}
  {#if gamePhase === 1}
  <div class="phase1">
    <p>We will now reveal what is behind one of the doors you did not select.</p>
    <button class = "btn" on:click={revealHost}>Reveal</button>
  </div>
  {/if}
  {#if gamePhase === 2}
  <div class="phase2">
    <p>Would you like to change your selection? Stay or Switch</p>
    <div class = "twoBtns">
      <button class = "btn" on:click={userStay}>Stay</button>
      <button class = "btn" on:click={userSwitch}>Switch</button>
    </div>
  </div>
  {/if}
  {#if gamePhase === 3}
  <div class="phase3">
    <p>See if you've won!</p>
    <button class = "btn" on:click={revealAll}>Reveal</button>
  </div>
  {/if}
  {#if gamePhase === 4}
  <div style="padding-top: 20px;">
    <p>{gameResult}</p>
    <div class = "twoBtns">
      <button class = "btn" on:click={restartGame}>Try again</button>
      <button class = "btn">Tell me more!</button>
    </div>
  </div>
  {/if}
  <br>
  <p>Would you believe it if we told you that choosing not to switch your selection after the host reveals a door to you decreases your probability of winning to only <b>33%</b>?</p>
  <p>Don't believe us? Click the button below to simulate 100 games where the user stays.</p>
  <button class = "btn" on:click={() => { 
    for (let i = 0; i < 100; i++) {
      setTimeout(() => autoselectStay(i), i * 500); 
      }
    }}>
    Simulate 100 Games
  </button>
  <div class = "simulation">
    <div class="row">
      <div class = "column" on:click={() => selectDoor(0)}
        style={`border: solid 5px ${userSelection === 0 ? "black" : "#fef2cf"}`}
        >
        <img class = "door" src= "closed-door.svg"/>
      </div>
      <div class = "column" on:click={() => selectDoor(1)}
        style={`border: solid 5px ${userSelection === 1 ? "black" : "#fef2cf"}`}
        >
        <img class = "door"  src= "closed-door.svg"/>
      </div>
      <div class = "column" on:click={() => selectDoor(2)}
        style={`border: solid 5px ${userSelection === 2 ? "black" : "#fef2cf"}`}
        >
        <img class = "door" src= "closed-door.svg"/>
      </div>
      <div class = "column" id="lineChart"></div>
    </div>
  </div>
  <br>
  <br>
  <p>As you might have noticed, the car wasn't chosen very often when we don't change our selection. We are converging towards a 33% win rate. Those don't seem like good odds!</p>
  <div id="lineChart"></div>
  <!-- <img src= "switch-graph-png.png"/> -->
  <br>
  <br>
  <br>
  <button class="btn">What if we switch?</button>
  <br>
  <br>
  <br>
  <br>
  <p>Based on fancy math (which we will explain later), switching gives us a higher win rate of <b>66%</b>!</p>
  <p>Don't take our word for it though. Simulate 100 games where the user switches.</p>
  <button class = "btn" on:click={() => { 
    for (let i = 0; i < 100; i++) {
      setTimeout(() => autoselectSwitch(i), i * 500);
      }
    }}>
    Simulate 100 Games
  </button>
  <div class = "simulation">
    <div class="row">
      <div class = "column" on:click={() => selectDoor(0)}
        style={`border: solid 5px ${userSelection === 0 ? "black" : "#fef2cf"}`}
        >
        <img class = "door" src= "closed-door.svg"/>
      </div>
      <div class = "column" on:click={() => selectDoor(1)}
        style={`border: solid 5px ${userSelection === 1 ? "black" : "#fef2cf"}`}
        >
        <img class = "door"  src= "closed-door.svg"/>
      </div>
      <div class = "column" on:click={() => selectDoor(2)}
        style={`border: solid 5px ${userSelection === 2 ? "black" : "#fef2cf"}`}
        >
        <img class = "door" src= "closed-door.svg"/>
      </div>
      <div class = "column" id="lineChartSwitch"></div>
    </div>
  </div>
  <br>
  <br>
  <br>
  <br>
  <p>That's more like it! As you can see, there's a higher probability of winning when the user switches their selection insteading of staying on their original door of choice.</p>
  <!-- <img src= "stay-graph-png.png"/> -->
  <br>
  <br>
  <button class = "btn">I'm not convinced!</button>
  <br>
  <br>
  <br>
  <br>
  <p><b>But that's only a sample size of 10, which isn't enough to draw conclusions. </b>Not to worry! We went ahead and generated hundreds of simulations of the Monty Hall problem.</p>
  <p>Believe us now? Switching is by far the best choice to optimze your chances at winning.</p>
  <div class="statistics">
    <table>
      <tr>
        <th>Games</th>
        <th>Stay</th>
        <th>Switch</th>
      </tr>
      <tr>
        <td>Wins:</td>
        <td>{score.stay.win}</td>
        <td>{score.switch.win}</td>
      </tr>
      <tr>
        <td>Losses:</td>
        <td>{score.stay.loss}</td>
        <td>{score.switch.loss}</td>
      </tr>
      <tr>
        <td>Win Statistic:</td>
        <td>{Math.round((score.stay.win / (score.stay.win + score.stay.loss)) * 100) || 0 }%</td>
        <td>{Math.round((score.switch.win / (score.switch.win + score.switch.loss)) * 100) || 0}%</td>
      </tr>
    </table>
  </div>
  <div class = "pieCharts">
    <div class = "pieChart" id="pieChartStay">Win-Loss Breakdown for Staying<br></div>
    <div class = "pieChart" id="pieChartSwitch">Win-Loss Breakdown for Switching<br></div>
  </div>
  <button class = "btn">Show me the math</button>
  <br>
  <br>
  <center>
  <p>Probabilities determined based on the following arrangement of car and goats:</p>
  <table border="1">
    <tr>
        <th>Door 1</th>
        <th>Door 2</th>
        <th>Door 3</th>
    </tr>
    <tr>
        <td>goat</td>
        <td>goat</td>
        <td>car</td>
    </tr>
</table>
  <h2>Outcomes for Staying</h2>
  <table border="1">
      <tr>
          <th>Door 1</th>
          <th>Door 2</th>
          <th>Door 3</th>
          <th>Strategy</th>
          <th>Outcome</th>
      </tr>
      <tr>
          <td>goat</td>
          <td>goat</td>
          <td>car</td>
          <td>Pick Door 1</td>
          <td>Lose</td>
      </tr>
      <tr>
          <td>goat</td>
          <td>goat</td>
          <td>car</td>
          <td>Pick Door 2</td>
          <td>Lose</td>
      </tr>
      <tr>
          <td>goat</td>
          <td>goat</td>
          <td>car</td>
          <td>Pick Door 3</td>
          <td>Win</td>
      </tr>
  </table>

  <h2>Outcomes for Switching</h2>
  <table border="1">
      <tr>
          <th>Door 1</th>
          <th>Door 2</th>
          <th>Door 3</th>
          <th>Strategy</th>
          <th>Outcome</th>
      </tr>
      <tr>
          <td>goat</td>
          <td>goat</td>
          <td>car</td>
          <td>Pick door 1. Host shows Door 2. You switch to Door 3</td>
          <td>Win</td>
      </tr>
      <tr>
          <td>goat</td>
          <td>goat</td>
          <td>car</td>
          <td>Pick door 2. Host shows Door 1. You switch to Door 3</td>
          <td>Win</td>
      </tr>
      <tr>
          <td>goat</td>
          <td>goat</td>
          <td>car</td>
          <td>Pick door 3. Host shows Door 1 or 2. Regardless, you switch</td>
          <td>Lose</td>
      </tr>
  </table>
</center>
  <p><b>Bayes' Theorem: </b>P(A|B) = P(B|A)P(A) / P(B) = P(B|A) * P(A) / P(B|A) * P(A) + P(B|not A) * P(not A)</p>
  <p><b>Probability of winning car given staying at original door: </b>(1/2)(1/3) / ((1/2)(1/3) + (0)(1/3) + (1)(1/3)) = 1/3</p>
  <p><b>Probability of winning car given switching doors: </b>(1)(1/3) / ((1/2)(1/3) + (0)(1/3) + (1)(1/3)) = 2/3</p>
  <br>
  <br>
  <br>
  <br>
  <br>
  <br>
  <br>
  <br>
</main>

<style>
  .btn {
    display: block;
    position: relative;
    /* bottom: 7rem; */
    left: 50%;
    width: 15rem;
    -webkit-transform: translateX(-50%);
    -ms-transform: translateX(-50%);
    transform: translateX(-50%);
    background-color: #fff;
    font-weight: 600;
    border: 1px solid rgba(2, 35, 64, .25);
    -webkit-transition: all 150ms ease-in-out;
    -o-transition: all 150ms ease-in-out;
    transition: all 150ms ease-in-out;
    cursor: pointer;
    background-color: #fce303;
    opacity: 1;
    -webkit-box-shadow: 0 2px 4px rgba(2, 35, 64, .33);
    box-shadow: 0 2px 4px rgba(2, 35, 64, .33);
    border-radius: 2rem;
    color: #022340;
    padding: 0.35rem 0;
    font-size: 1rem;
  }
  .btn:hover {
    background-color: #c3af00;
  }
  .twoBtns {
    display: flex;
    justify-content: center;
  }
  .twoBtns .btn{
    left:10%;
    margin:3%;
  }
  main {
    font-family: 'Barlow';
    text-align: center;
    /* padding: 1em; */
    /* max-width: 240px; */
    width: 100%;
    height: auto;
    padding: 0;
    margin: 0;
    background-color: #fef2cf;
  }
  p {
    font-size: 22px;
  }
  .title-page img{
    width: 100%;
    height:auto;
  }
  .title-page .aboutBtn {
    position: fixed;
    top: 0.75rem;
    right: 0.75rem;
    background-color: #fce303;
    border: 1px solid rgba(2, 35, 64, .25);
    color: #022340;
    -webkit-box-shadow: 0 2px 4px rgba(2, 35, 64, .33);
    box-shadow: 0 2px 4px rgba(2, 35, 64, .33);
    border-radius: 50%;
    width: 1.5rem;
    height: 1.5rem;
    font-weight: 600;
    z-index: 1000;
  }
  .title-page .btn {
    display: block;
    position: relative;
    bottom: 7rem;
    left: 50%;
    width: 15rem;
    -webkit-transform: translateX(-50%);
    -ms-transform: translateX(-50%);
    transform: translateX(-50%);
    background-color: #fff;
    font-weight: 600;
    border: 1px solid rgba(2, 35, 64, .25);
    -webkit-transition: all 150ms ease-in-out;
    -o-transition: all 150ms ease-in-out;
    transition: all 150ms ease-in-out;
    cursor: pointer;
    background-color: #fce303;
    opacity: 1;
    -webkit-box-shadow: 0 2px 4px rgba(2, 35, 64, .33);
    box-shadow: 0 2px 4px rgba(2, 35, 64, .33);
    border-radius: 2rem;
    color: #022340;
    padding: 0.35rem 0;
    font-size: 1rem;
  }
  .title-page .btn:hover {
    background-color: #c3af00;
  }
  .title-page .aboutBtn:hover {
    background-color: #c3af00;
  }
  .row img {
    width: 100%;
    height: auto;
  }
  .row {
    display: flex;
    width:70%;
    padding-left: 15%;
    justify-content: center;
  }

  .simulation .row {
    display: flex;
    width:70%;
    justify-content: center;
    align-items: center;
  }
  .simulation .column {
    flex: 20%;
    padding: 5px;
  }
  .column {
    flex: 33.33%;
    padding: 5px;
  }
  .pieCharts {
    display: flex;
    width:80%;
    padding-left: 10%;
    justify-content: center;
  }

  .pieChart {
    flex: 33.33%;
    padding: 5px;
  }
  .statistics {
    display: flex;
    justify-content: center;
    padding: 20px
  }
  h1 {
    color: #000000;
    font-size: 3em;
    font-weight: 100;
  }
  h2 {
    color: #000000;
    font-size: 1.5em;
    font-weight: 50;
  }
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
  table {
    border-collapse: collapse;
  }
  td, th {
    padding: 8px;
  }
</style>