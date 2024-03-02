<script>
  let score
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
  const doorColor = {
          lose: "grey",
          empty: "red",
          full: "yellowgreen",
  };
  let bgColor = []; function initialize() {
          gamePhase = 0;
          gameResult = "";
          prizeDoor = selectPrizeDoor();
          userSelection = null;
          bgColor[0] = doorColor.close;
          bgColor[1] = doorColor.close;
          bgColor[2] = doorColor.close;
  }
  initialize(); function openDoor(doorNumber) {
        bgColor[doorNumber] =
                  prizeDoor === doorNumber ? doorColor.full : doorColor.empty;
  } function selectDoor(doorNumber) {
      if (gamePhase === 0) {
              userSelection = doorNumber;
      }
  } function restartGame() {
      initialize();
  } function revealHost() {
      if (gamePhase === 0 && userSelection !== null) {
          gamePhase = 1;
          if (userSelection === prizeDoor) {
              const doors = [0, 1, 2];
              doors.splice(userSelection, 1);
              let randomBinary = new Date();
              randomBinary = randomBinary.getMilliseconds() % 2;
              console.log("randomBinary", randomBinary);
              hostRevealed = doors[randomBinary];
          } 
          else {
              hostRevealed = 3 - userSelection - prizeDoor;
              // 3 - 0 - 1 - 2 = 0
              // 3 - 0 - 1 = 2
              // 3 - 0 - 2 = 1
              // 3 - 1 - 2 = 0
          }
          openDoor(hostRevealed);
    }
  } function userStay() {
      if (gamePhase === 1) {
          gamePhase = 2;
          path = "stay";
      }
  } function userSwitch() {
      if (gamePhase === 1) {
          userSelection = 3 - userSelection - hostRevealed;
          gamePhase = 2;
          path = "switch";
      }
  } function revealAll() {
      if (gamePhase === 2) {
          gamePhase = 3;
          openDoor(0);
          openDoor(1);
          openDoor(2);
          gameResult = userSelection === prizeDoor ? "WIN" : "LOSE";
          score[path][["loss", "win"][Number(userSelection === prizeDoor)]]++;
          localStorage.setItem("score", JSON.stringify(score));
      }
  }
</script>

<main>
  <h1><b>Welcome to the DSC 106 Game Show!<b></h1>
  <h2>An interactive experiment of the <b>Monty Hall Paradox<b></h2>
  <p style="margin-left: auto; margin-right: auto; width: 66em; font-size: 18px">Suppose you're on a game show. You are given the choice of three doors: Behind one door is a grand prize of a car; behind the others, goats.</p>
  <div class="phase0"
      style={`background-color:${gamePhase === 0 && userSelection === null ? "#cfc" : "white"}`}
  >
      <p>Pick a door! Any door!</p>
      <div class="doors">
        <div class="door" on:click={() => selectDoor(0)}
            style={`background-color:${bgColor[0]}; border: solid 5px ${
              userSelection === 0 ? "black" : "white"
            }`}
        >
        door 1
        </div>
        <div class="door" on:click={() => selectDoor(1)}
            style={`background-color:${bgColor[1]}; border: solid 5px ${
              userSelection === 1 ? "black" : "white"
            }`}
        >
        door 2
        </div>
        <div class="door" on:click={() => selectDoor(2)}
            style={`background-color:${bgColor[2]}; border: solid 5px ${
                userSelection === 2 ? "black" : "white"
            }`}
        >
        door 3
        </div>
      </div>
  </div>
  <div class="phase1"
      style={`background-color:${gamePhase === 0 && userSelection !== null ? "#cfc" : "white"}`}
  >
      <p>The host will now reveal what is behind one of the doors</p>
      <button on:click={revealHost}>Reveal</button>
  </div>
  <div class="phase2"
      style={`background-color:${gamePhase === 1 ? "#cfc" : "white"}`}
  >
      <p>Would you like to change your selection? Stay or Switch</p>
      <button on:click={userStay}>Stay</button>
      <button on:click={userSwitch}>Switch</button>
  </div>
  <div class="phase3"
      style={`background-color:${gamePhase === 2 ? "#cfc" : "white"}`}
  >
      <p>See if you've won!</p>
      {gameResult} <button on:click={revealAll}>Reveal</button>
      {gameResult}
  </div>
  <div style="padding-top: 20px;"> </div>
  <hr />
  <div style="padding-top: 20px;">
    <button on:click={restartGame}>New Game</button>
  </div>
  <div class="statistics">
    <table >
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
  <div>
    <p><b>Is it better to stay or switch?</b></p>
    <p>The answer can be revealed through the Bayes' theorem. Breaking down the equation for Bayes' theorem even further, P(A|B) = P(B|A)P(A) / P(B) = P(B|A) * P(A) / P(B|A) * P(A) + P(B|not A) * P(not A). In this case, P(A) is the marginal probability that door 1 has a car behind it without knowing about the door that the host reveals, so it is equal to 1/3. P(not A) is the probability that the player did not pick the door with the car behind it. Since P(not A) is the probability that the player does not pick the door with the car behind it. Since the door either has the car behind it or not, P(B) is equal to 1 - P(A), 2/3. P(B|A) is the probability that the host shows a door with a goat behind it, given that there is a car behind door 1. Since the host always shows a door with a goat, P(B|A) is equal to 1. P(B|not A) is the probability that the host shows the goat, given that there is a goat behind door 1. As mentioned previously like P(B|A), the host always shows a door with a goat, so P(B|not A) is also equal to 1. Plugging each number into the equation gets us (1)(1/3) / ((1)(1/3) + (1)(2/3)) = (1/3) / 1 = 1/3. The probability that the car is behind the door of the player's choice is unchanged by the evidence. However, since the car can only either be behind the door of the player's choice or behind the door the host didn't reveal, the probability it is behind the door that is not revealed is 2/3. Therefore, switching is twice as likely to get you the prize as staying.</p>
  </div>
  </main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }.doors {
    display: flex;
    justify-content: center;
    margin: 5px;
  }.door {
    border: solid 1px;
    width: 80px;
    height: 160px;
    margin: 5px;
    padding: 10px;
  }.statistics {
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
</style>