<script>
  import * as d3 from "d3";
  import { line, scaleLinear, axisBottom, axisLeft } from 'd3';
  import Katex from 'svelte-katex'
  
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
  let simStayScoreWin = 0;
  let simStayScoreLoss = 0;
  let simSwitchScoreWin =  0;
  let simSwitchScoreLoss =  0;
  let path = ""; // "stay" | "switch"
  let gamePhase = 0;
  let userSelection = null;
  let hostRevealed = null;
  let gameResult = "";
  let pageNumber = 0;
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
    setTimeout(() => {  revealHost(); }, 50);
    setTimeout(() => {  userStay(); }, 50);
    setTimeout(() => {  revealAllSim(); }, 100);
    setTimeout(() => {  data.push({ trial: iteration+1, percentage: Math.round((simStayScoreWin / (simStayScoreWin + simStayScoreLoss)) * 100) });  }, 100);
    setTimeout(() => {  drawChart(); }, 150);
    setTimeout(() => {  restartGame(); }, 200);
  }
  function autoselectSwitch(iteration) {
    userSelection = Math.floor(Math.random() * 3);
    setTimeout(() => {  revealHost(); }, 50);
    setTimeout(() => {  userSwitch(); }, 75);
    setTimeout(() => {  revealAllSim(); }, 100);
    setTimeout(() => {  dataSwitch.push({ trial: iteration+1, percentage: Math.round((simSwitchScoreWin / (simSwitchScoreWin + simSwitchScoreLoss)) * 100) });  }, 100);
    setTimeout(() => {  drawLineChart(); }, 150);
    setTimeout(() => {  restartGame(); }, 200);
  }
  function restartGame() {
    closeDoor(0);
    closeDoor(1);
    closeDoor(2);
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
      // openDoor(hostRevealed+3);
      // openDoor(hostRevealed+6);
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
  function revealAllSim() {
    openDoor(0);
    openDoor(1);
    openDoor(2);
    if (path === 'stay') {
      if (userSelection === prizeDoor) {
        simStayScoreWin++;
      } else {
        simStayScoreLoss++;
      }
    } else {
      if (userSelection === prizeDoor) {
        simSwitchScoreWin++;
      } else {
        simSwitchScoreLoss++;
      }
    }

  }
  function revealAllGame() {
    openDoor(0);
    openDoor(1);
    openDoor(2);
    if (userSelection === prizeDoor) {
      gameResult = "You won!  But did you optimize your odds or were you just lucky? In other words, is it better to stay or to switch?"
    } else {
      gameResult = "You lost! Is there a way to optimize your chance for next time? In other words, is it better to stay or to switch?"
    }
    gamePhase++;
  }
  function revealAll() {
    openDoor(0);
    openDoor(1);
    openDoor(2);
    if (userSelection === prizeDoor) {
      gameResult = "You won!  But did you optimize your odds or were you just lucky? In other words, is it better to stay or to switch?"
    } else {
      gameResult = "You lost! Is there a way to optimize your chance for next time? In other words, is it better to stay or to switch?"
    }
    score[path][["loss", "win"][Number(userSelection === prizeDoor)]]++;
    localStorage.setItem("score", JSON.stringify(score));
    if (path==="stay") {
      updatePieChartStay();
    } else {
      updatePieChartSwitch();
    }
    gamePhase++;
  }
  function updatePieChartStay() {
    d3.select('#pieChartStay').selectAll('svg').remove();
    const data = [
        { label: 'Win', value: score.stay.win },
        { label: 'Loss', value: score.stay.loss }
    ];
    const width = 150;
    const height = 150;
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
        .outerRadius(radius - 25)
        .innerRadius(radius - 25);

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
        .text(d => (Math.round(d.data.value/(score.stay.win + score.stay.loss) * 100) > 0) ? Math.round(d.data.value/(score.stay.win + score.stay.loss) * 100)+"%" : '')
        .style("text-anchor", "middle")
        .style("font-size", 12)
        .style('font-family', 'Barlow');
}
function updatePieChartSwitch() {
  d3.select('#pieChartSwitch').selectAll('svg').remove();
    const data = [
        { label: 'Win', value: score.switch.win },
        { label: 'Loss', value: score.switch.loss }
    ];
    const width = 150;
    const height = 150;
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
        .outerRadius(radius - 25)
        .innerRadius(radius - 25);

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
        .text(d => (Math.round(d.data.value/(score.switch.win + score.switch.loss) * 100) > 0) ? Math.round(d.data.value/(score.switch.win + score.switch.loss) * 100)+"%" : '')
        .style("text-anchor", "middle")
        .style("font-size", 12)
        .style('font-family', 'Barlow');
}
let data = [
    ];
let dataSwitch = [
    ];

function drawChart() {
  d3.select('#lineChart').selectAll('svg').remove();
      const margin = { top: 50, right: 20, bottom: 50, left: 100 };
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
      const yAxis = axisLeft(y).ticks(4);

      const gridlines = svg.append('g')
        .attr('class', 'grid')
        .attr('color', '#dec8b1')
        .style("stroke-dasharray", ("3, 3")) 
        .call(axisLeft(y).tickSize(-width).tickFormat(''));

      // x-axis nums
      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(xAxis)
        .style('font-size', '15px');

      // y-axis nums
      svg.append('g')
        .call(yAxis.tickFormat(d => d + '%'))
        .style('font-size', '15px');

      // thicker x and y axis lines
      svg.selectAll('.domain')
	      .style('stroke-width', '2px');

      const lineGenerator = line()
        .x(d => x(d.trial))
        .y(d => y(d.percentage));

      const path = svg.append('path')
        .datum(data)
        .attr('fill', 'none')
        .attr('stroke', 'red')
        .attr('stroke-width', 1.5)
        .attr('d', lineGenerator);
      // top right num of successes
      const numSuccesses = svg.append("text")
        .attr("y", 15)
        .attr("x", 340)
        .attr('text-anchor', 'middle')
        .attr("class", "successes")
        .attr("font-weight", 600)
        .html(function() {
	        return `<tspan fill="#dec8b1">${simStayScoreWin}</tspan><tspan fill="#dec8b1"> SUCCESSES</tspan>`;
				});

      // bottom right num of failures
      const numFailures = svg.append("text")
        .attr("y", 195)
        .attr("x", 350)
        .attr('text-anchor', 'middle')
        .attr("class", "failures")
        .attr("font-weight", 600)
        .html(function() {
	        return `<tspan fill="#dec8b1">${simStayScoreLoss}</tspan><tspan fill="#dec8b1"> FAILURES</tspan>`;
				});

      // x-axis label
      svg.append("text")
        .attr("class", "x label")
        .attr("text-anchor", "middle")
        .attr("x", 200)
        .attr("y", 240)
        .attr("font-weight", 600)
        .text("NUMBER OF TRIALS");

      // y-axis label
      svg.append("text")
        .attr("class", "y label")
        .attr("text-anchor", "middle")
        .attr("y", -50)
        .attr("x", -100)
        .attr("font-weight", 600)
        .attr("transform", "rotate(-90)")
        .text("SUCCESS RATE");

  }
  function drawLineChart() {
    d3.select('#lineChartSwitch').selectAll('svg').remove();
      const margin = { top: 50, right: 20, bottom: 50, left: 100 };
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
      const yAxis = axisLeft(y).ticks(4);

      const gridlines = svg.append('g')
        .attr('class', 'grid')
        .attr('color', '#dec8b1')
        .style("stroke-dasharray", ("3, 3")) 
        .call(axisLeft(y).tickSize(-width).tickFormat(''));

      // x-axis nums
      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(xAxis)
        .style('font-size', '15px');

      // y-axis nums
      svg.append('g')
        .call(yAxis.tickFormat(d => d + '%'))
        .style('font-size', '15px');

      // thicker x and y axis lines
      svg.selectAll('.domain')
	      .style('stroke-width', '2px');

      const lineGenerator = line()
        .x(d => x(d.trial))
        .y(d => y(d.percentage));

      const path = svg.append('path')
        .datum(dataSwitch)
        .attr('fill', 'none')
        .attr('stroke', 'red')
        .attr('stroke-width', 1.5)
        .attr('d', lineGenerator);
      // top right num of successes
      const numSuccesses = svg.append("text")
        .attr("y", 15)
        .attr("x", 340)
        .attr('text-anchor', 'middle')
        .attr("class", "successes")
        .attr("font-weight", 600)
        .html(function() {
	        return `<tspan fill="#dec8b1">${simSwitchScoreWin}</tspan><tspan fill="#dec8b1"> SUCCESSES</tspan>`;
				});

      // bottom right num of failures
      const numFailures = svg.append("text")
        .attr("y", 195)
        .attr("x", 350)
        .attr('text-anchor', 'middle')
        .attr("class", "failures")
        .attr("font-weight", 600)
        .html(function() {
	        return `<tspan fill="#dec8b1">${simSwitchScoreLoss}</tspan><tspan fill="#dec8b1"> FAILURES</tspan>`;
				});

      // x-axis label
      svg.append("text")
        .attr("class", "x label")
        .attr("text-anchor", "middle")
        .attr("x", 200)
        .attr("y", 240)
        .attr("font-weight", 600)
        .text("NUMBER OF TRIALS");

      // y-axis label
      svg.append("text")
        .attr("class", "y label")
        .attr("text-anchor", "middle")
        .attr("y", -50)
        .attr("x", -100)
        .attr("font-weight", 600)
        // .attr("dy", ".75em")
        .attr("transform", "rotate(-90)")
        .text("SUCCESS RATE");
  }
  function nextPage(){
    pageNumber++;
  }
</script>

<main>
  {#if pageNumber === 0}
  <div class = "title-page">
    <img src = "title-page.svg"/>
    <button class = "btn" on:click={() => {nextPage();}}>Let's do this!</button>
  </div>
  {/if}
  {#if pageNumber === 1}
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
    <button class = "btn" on:click={revealAllGame}>Reveal</button>
  </div>
  {/if}
  {#if gamePhase === 4}
  <div style="padding-top: 20px;">
    <p>{gameResult}</p>
    <div class = "twoBtns">
      <button class = "btn" on:click={restartGame}>Try again</button>
      <button class = "btn" on:click={() => {restartGame(); nextPage();}}>Tell me more!</button>
    </div>
  </div>
  {/if}
  {/if}
  {#if pageNumber === 2}
  <div>
  <p>Would you believe it if we told you that choosing not to switch your selection after the host reveals a door to you decreases your probability of winning to only <b>33%</b>?</p>
  <p>Don't believe us? Click the button below to simulate 100 games where the user stays.</p>
  <button class="btn" on:click="{() => { 
      for (let i = 0; i < 100; i++) { 
        setTimeout(() => { 
          autoselectStay(i);
          document.querySelector('.btn').style.display = 'none';  
        }, i * 250); 
      } 
    }}">
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
      <div class = "column" id="lineChart">
        <svg width="520" height="306"><g transform="translate(100,50)"><g class="grid" color="#dec8b1" fill="none" font-size="10" font-family="sans-serif" text-anchor="end" style="stroke-dasharray: 3, 3;"><path class="domain" stroke="currentColor" d="M400,200H0V0H400" style="stroke-width: 2px;"></path><g class="tick" opacity="1" transform="translate(0,200)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,180)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,160)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,140)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,120)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,100)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,80)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,60.00000000000001)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,39.99999999999999)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,19.999999999999996)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,0)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g></g><g transform="translate(0,200)" fill="none" font-size="10" font-family="sans-serif" text-anchor="middle" style="font-size: 15px;"><path class="domain" stroke="currentColor" d="M0,6V0H400V6" style="stroke-width: 2px;"></path><g class="tick" opacity="1" transform="translate(36.36363636363637,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">10</text></g><g class="tick" opacity="1" transform="translate(76.76767676767676,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">20</text></g><g class="tick" opacity="1" transform="translate(117.17171717171718,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">30</text></g><g class="tick" opacity="1" transform="translate(157.57575757575756,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">40</text></g><g class="tick" opacity="1" transform="translate(197.97979797979798,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">50</text></g><g class="tick" opacity="1" transform="translate(238.38383838383837,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">60</text></g><g class="tick" opacity="1" transform="translate(278.7878787878788,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">70</text></g><g class="tick" opacity="1" transform="translate(319.1919191919192,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">80</text></g><g class="tick" opacity="1" transform="translate(359.5959595959596,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">90</text></g><g class="tick" opacity="1" transform="translate(400,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">100</text></g></g><g fill="none" font-size="10" font-family="sans-serif" text-anchor="end" style="font-size: 15px;"><path class="domain" stroke="currentColor" d="M-6,200H0V0H-6" style="stroke-width: 2px;"></path><g class="tick" opacity="1" transform="translate(0,200)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">0%</text></g><g class="tick" opacity="1" transform="translate(0,160)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">20%</text></g><g class="tick" opacity="1" transform="translate(0,120)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">40%</text></g><g class="tick" opacity="1" transform="translate(0,80)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">60%</text></g><g class="tick" opacity="1" transform="translate(0,39.99999999999999)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">80%</text></g><g class="tick" opacity="1" transform="translate(0,0)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">100%</text></g></g><text y="15" x="340" text-anchor="middle" class="successes" font-weight="600"><tspan fill="#dec8b1">0 </tspan><tspan fill="#dec8b1"> SUCCESSES</tspan></text><text y="195" x="350" text-anchor="middle" class="failures" font-weight="600"><tspan fill="#dec8b1">0 </tspan><tspan fill="#dec8b1"> FAILURES</tspan></text><text class="x label" text-anchor="middle" x="200" y="240" font-weight="600">NUMBER OF TRIALS</text><text class="y label" text-anchor="middle" y="-50" x="-100" font-weight="600" transform="rotate(-90)">SUCCESS RATE</text></g></svg>
      </div>
    </div>
  </div>
  <br>
  <br>
  <p>As you might have noticed, the car wasn't chosen very often when we stuck with our selection. We are converging towards a 33% win rate. Those don't seem like good odds!</p>
  <p style="font-size:18px;">Please click the button below after the line chart has finished plotting the data.</p>
  <button class="btn" on:click={nextPage}>What if we switch?</button>
</div>
  {/if}

  {#if pageNumber === 3}
  <p>Based on fancy math (which we will explain a bit later), switching gives us a higher win rate of <b>66%</b>!</p>
  <p>Don't take our word for it though. Simulate 100 games where the user switches.</p>
  <button class="btn" on:click="{() => { 
      for (let i = 0; i < 100; i++) { 
        setTimeout(() => { 
          autoselectSwitch(i);
          document.querySelector('.btn').style.display = 'none';  
        }, i * 250); 
      } 
    }}">
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
      <div class = "column" id="lineChartSwitch">
                <svg width="520" height="306"><g transform="translate(100,50)"><g class="grid" color="#dec8b1" fill="none" font-size="10" font-family="sans-serif" text-anchor="end" style="stroke-dasharray: 3, 3;"><path class="domain" stroke="currentColor" d="M400,200H0V0H400" style="stroke-width: 2px;"></path><g class="tick" opacity="1" transform="translate(0,200)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,180)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,160)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,140)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,120)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,100)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,80)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,60.00000000000001)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,39.99999999999999)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,19.999999999999996)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g><g class="tick" opacity="1" transform="translate(0,0)"><line stroke="currentColor" x2="400"></line><text fill="currentColor" x="-3" dy="0.32em"></text></g></g><g transform="translate(0,200)" fill="none" font-size="10" font-family="sans-serif" text-anchor="middle" style="font-size: 15px;"><path class="domain" stroke="currentColor" d="M0,6V0H400V6" style="stroke-width: 2px;"></path><g class="tick" opacity="1" transform="translate(36.36363636363637,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">10</text></g><g class="tick" opacity="1" transform="translate(76.76767676767676,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">20</text></g><g class="tick" opacity="1" transform="translate(117.17171717171718,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">30</text></g><g class="tick" opacity="1" transform="translate(157.57575757575756,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">40</text></g><g class="tick" opacity="1" transform="translate(197.97979797979798,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">50</text></g><g class="tick" opacity="1" transform="translate(238.38383838383837,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">60</text></g><g class="tick" opacity="1" transform="translate(278.7878787878788,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">70</text></g><g class="tick" opacity="1" transform="translate(319.1919191919192,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">80</text></g><g class="tick" opacity="1" transform="translate(359.5959595959596,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">90</text></g><g class="tick" opacity="1" transform="translate(400,0)"><line stroke="currentColor" y2="6"></line><text fill="currentColor" y="9" dy="0.71em">100</text></g></g><g fill="none" font-size="10" font-family="sans-serif" text-anchor="end" style="font-size: 15px;"><path class="domain" stroke="currentColor" d="M-6,200H0V0H-6" style="stroke-width: 2px;"></path><g class="tick" opacity="1" transform="translate(0,200)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">0%</text></g><g class="tick" opacity="1" transform="translate(0,160)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">20%</text></g><g class="tick" opacity="1" transform="translate(0,120)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">40%</text></g><g class="tick" opacity="1" transform="translate(0,80)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">60%</text></g><g class="tick" opacity="1" transform="translate(0,39.99999999999999)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">80%</text></g><g class="tick" opacity="1" transform="translate(0,0)"><line stroke="currentColor" x2="-6"></line><text fill="currentColor" x="-9" dy="0.32em">100%</text></g></g><text y="15" x="340" text-anchor="middle" class="successes" font-weight="600"><tspan fill="#dec8b1">0 </tspan><tspan fill="#dec8b1"> SUCCESSES</tspan></text><text y="195" x="350" text-anchor="middle" class="failures" font-weight="600"><tspan fill="#dec8b1">0 </tspan><tspan fill="#dec8b1"> FAILURES</tspan></text><text class="x label" text-anchor="middle" x="200" y="240" font-weight="600">NUMBER OF TRIALS</text><text class="y label" text-anchor="middle" y="-50" x="-100" font-weight="600" transform="rotate(-90)">SUCCESS RATE</text></g></svg>
      </div>
    </div>
  </div>

  <p>That's more like it! As you can see, there's a higher probability of winning when the user switches their selection insteading of staying on their original door of choice.</p>
  <p style="font-size:18px;">Please click the button below after the line chart has finished plotting the data.</p>
  <button class = "btn" on:click={nextPage}>I'm not convinced!</button>
  {/if}

  {#if pageNumber === 4}
  <p>Maybe you're not fully certain that switching is the better choice since we only ran 100 simulations each. Therefore, let's take a look at this problem through the lens of probability.</p>
  <p>Let's pretend that an instance of the game has the following arrangement of car and goats. Once you understand the probabilities and outcomes for one arrangement, you'll be able to apply the same concepts to understanding all different types of arrangements.</p>
  <div class="row">
    <div class = "column">
      <p>Door 1</p>
      <img class = "door" src= "goat.svg"/>
    </div>
    <div class = "column">
      <p>Door 2</p>
      <img class = "door"  src= "goat.svg"/>
    </div>
    <div class = "column">
      <p>Door 3</p>
      <img class = "door" src= "car.svg"/>
    </div>
  </div><center>

  
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
          <td>Goat</td>
          <td>Goat</td>
          <td>Car</td>
          <td>Pick Door 1</td>
          <td>Lose 🐐</td>
      </tr>
      <tr>
          <td>Goat</td>
          <td>Goat</td>
          <td>Car</td>
          <td>Pick Door 2</td>
          <td>Lose 🐐</td>
      </tr>
      <tr>
          <td>Goat</td>
          <td>Goat</td>
          <td>Car</td>
          <td>Pick Door 3</td>
          <td>Win 🚗</td>
      </tr>
  </table>
  <p style="font-size:20px;"><b>Win Percentage: </b>1 out of 3, or 33.33%</p>
<br>
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
          <td>Goat</td>
          <td>Goat</td>
          <td>Car</td>
          <td>Pick door 1. Host shows Door 2. You switch to Door 3</td>
          <td>Win 🚗</td>
      </tr>
      <tr>
          <td>Goat</td>
          <td>Goat</td>
          <td>Car</td>
          <td>Pick door 2. Host shows Door 1. You switch to Door 3</td>
          <td>Win 🚗</td>
      </tr>
      <tr>
          <td>Goat</td>
          <td>Goat</td>
          <td>Car</td>
          <td>Pick door 3. Host shows Door 1 or 2. Regardless, you switch</td>
          <td>Lose 🐐</td>
      </tr>
  </table>
  <p style="font-size: 18px;">Note: Remember that the door the host reveals is <i>always</i> a goat.</p>
  <p style="font-size:20px;"><b>Win Percentage: </b>2 out of 3, or 66.67%</p>
</center>
<br>
  <p>Believe us now? Switching is by far the best choice to optimze your chances at winning.</p>

<button class = "btn" on:click={nextPage}>Apply what you learned!</button>
<br>
<br>
<br>
{/if}
{#if pageNumber === 5}
<div class = "simulation">
  <p>Play the Monty Hall problem simulation as many times as you would like and see what your success rates are when you choose to "stay" or "switch".</p>
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
    <button class = "btn" on:click={restartGame}>Try again</button>
</div>
{/if}
<div class = "simulation">
  <br>
  <p style="font-size: 25px;">Your Win-Loss Statistics</p>
  <p style="font-size: 20px;">As you simulate games, your wins will be presented green and your losses will be presented red on the pie charts below.</p>
  <div class="row">
    <div>
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
          <td>Success Rate:</td>
          <td>{Math.round((score.stay.win / (score.stay.win + score.stay.loss)) * 100) || 0 }%</td>
          <td>{Math.round((score.switch.win / (score.switch.win + score.switch.loss)) * 100) || 0}%</td>
        </tr>
      </table>
    </div>
  <div class = "pieChart" id="pieChartStay">Win-Loss Breakdown (Stay)<br>
    <svg width="150" height="150"><g transform="translate(75,75)" stroke="#ECECEC" style="stroke-width: 1px;"><g class="arc"><path d="M0,-65A65,65,0,1,1,-60.173,24.581L0,0Z" fill="#ECECEC"></path></g><g class="arc"><path d="M-60.173,24.581A65,65,0,0,1,0,-65L0,0Z" fill="#ECECEC"></path></g></g></svg>
  </div>
  <div class = "pieChart" id="pieChartSwitch">Win-Loss Breakdown (Switch)<br>
    <svg width="150" height="150"><g transform="translate(75,75)" stroke="#ECECEC" style="stroke-width: 1px;"><g class="arc"><path d="M0,-65A65,65,0,1,1,-60.173,24.581L0,0Z" fill="#ECECEC"></path></g><g class="arc"><path d="M-60.173,24.581A65,65,0,0,1,0,-65L0,0Z" fill="#ECECEC"></path></g></g></svg>
  </div>
  </div>
</div>
<br>
<br>
<p style="font-size: 25px;"><b>Thank you for visiting the DSC 106 Game Show! Come back and play again!</b></p>
<span><a href="https://github.com/sophifang/bayes-theorem/tree/main" target="_blank">GitHub Repo</a>
    |  <a href="https://youtu.be/tQd_scXdlR4" target="_blank">Demo Video</a>
  </span>
<br>
<br>
<br>
{/if}
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

  .pieChart {
    flex: 33.33%;
    padding: 5px;
  }
  .statistics {
    display: flex;
    justify-content: center;
    padding: 20px
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
    text-align: center;
  }
</style>