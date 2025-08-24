<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Dot Game with DSA</title>
  <style>
    :root {
      --size: 400px;
    }

    * {
      margin: 0;
      padding: 0;
      border: 0;
      box-sizing: border-box;
    }

    html, body {
      height: 100%;
    }

    body {
      font-family: sans-serif;
      background: #2c3e50;
    }

    .info {
      position: fixed;
      top: 0;
      left: 0;
      background: #ecf0f1;
      border-radius: 0 0 5px 0;
      padding: 20px;
      color: #000;
    }

    .info .time::before {
      content: "Time left:";
      margin-right: 5px;
    }

    .info .points::before {
      content: "Points:";
      margin-right: 20px;
    }

    .wall {
      width: var(--size);
      height: var(--size);
      position: absolute;
      top: 0;
      left: 0;
      bottom: 0;
      right: 0;
      margin: auto;
    }

    .wall .dot {
      height: 10%;
      width: 10%;
      background: #ecf0f1;
      border-radius: 50%;
      float: left;
    }

    .wall .dot.active {
      background: #e74c3c;
      cursor: pointer;
    }

    .wall .start,
    .wall .end {
      position: absolute;
      height: 100%;
      width: 100%;
      background: #ecf0f1;
      border-radius: 5px;
      z-index: 100;
      top: 0;
      left: 0;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .wall .start button,
    .wall .end button {
      width: calc(var(--size) / 2);
      height: calc(var(--size) / 10);
      font-size: calc(var(--size) / 20);
      text-transform: uppercase;
      background: #e74c3c;
      border-radius: 5px;
      cursor: pointer;
      color: #fff;
    }

    .wall .start button:hover,
    .wall .end button:hover {
      background: #b13b2f;
    }

    .wall .end {
      display: none;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      text-align: center;
    }

    .wall .end .score {
      color: #000;
      font-size: calc(var(--size) / 15);
      margin-bottom: 20px;
    }

    .wall .end .highscores {
      color: #333;
      font-size: 16px;
      margin-top: 10px;
    }

    .wall::after {
      content: "";
      clear: both;
      display: table;
    }
  </style>
</head>
<body>
  <div class="info">
    <div class="time"></div>
    <div class="points"></div>
  </div>

  <div class="wall">
    <div class="start">
      <button>Start game!</button>
    </div>
    <div class="end">
      <div class="score"></div>
      <div class="highscores"></div>
      <button>Play again!</button>
    </div>
  </div>

  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <script>
    $(document).ready(function() {
      for (let i = 0; i < 100; i++) {
        $(".wall").append('<div class="dot" data-index="'+i+'"></div>');
      }

      $(document).on("click", ".start button, .end button", function() {
        App.start_game();
      });

      $(document).on("click", ".dot.active", function() {
        App.add_point($(this).data("index"));
      });
    });

    var App = {
      dot_selector: ".wall .dot",
      points: 0,
      time: 30,
      time_interval: null,
      availableIndices: [],
      usedIndices: new Set(),
      dotClickCounts: new Map(),
      highScores: [],

      init_game: function() {
        this.points = 0;
        this.time = 30;
        this.availableIndices = [...Array(100).keys()];
        this.shuffle(this.availableIndices);
        this.usedIndices.clear();
      },

      start_game: function() {
        this.init_game();
        $(".start, .end").fadeOut("fast");
        this.set_dot();
        $(".info .time").html("00:" + this.time);
        $(".info .points").html(this.points);
        this.time_interval = setInterval(() => {
          App.update_time();
        }, 1000);
      },

      update_time: function() {
        this.time -= 1;
        if (this.time > 0) {
          let timeStr = this.time < 10 ? "0" + this.time : this.time;
          $(".info .time").html("00:" + timeStr);
        } else {
          $(".info .time").html("00:00");
          clearInterval(this.time_interval);
          this.highScores.push(this.points);
          this.highScores.sort((a,b)=>b-a);
          if (this.highScores.length > 5) this.highScores.pop();
          $(".end .score").html(
            "Game over!<br />You scored " + this.points + " points!"
          );
          $(".end .highscores").html(
            "<strong>High Scores:</strong><br>" +
            this.highScores.join("<br>")
          );
          $(".end").fadeIn("fast");
        }
      },

      add_point: function(dotIndex) {
        this.points += 1;
        $(".info .points").html(this.points);

        if (this.dotClickCounts.has(dotIndex)) {
          this.dotClickCounts.set(dotIndex, this.dotClickCounts.get(dotIndex) + 1);
        } else {
          this.dotClickCounts.set(dotIndex, 1);
        }

        this.set_dot();
      },

      set_dot: function() {
        $(this.dot_selector).removeClass("active");

        if (this.availableIndices.length === 0) {
          this.availableIndices = [...this.usedIndices];
          this.usedIndices.clear();
          this.shuffle(this.availableIndices);
        }

        let index = this.availableIndices.pop();
        this.usedIndices.add(index);
        $(this.dot_selector).eq(index).addClass("active");
      },

      shuffle: function(array) {
        for (let i = array.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [array[i], array[j]] = [array[j], array[i]];
        }
      }
    };
  </script>
</body>
</html>
