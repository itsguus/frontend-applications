<template>
  <div id="chart">
    <myData />

    <section class="visualisation">
      <h1>Beschikbare parkeerplaatsen</h1>
      <h2><span class="day">MAANDAG</span> <span class="time">12:00</span></h2>
      <div class="percentage">
        <input type="checkbox" />
        <span>Aantallen</span>
        <div></div>
        <span>Percentages</span>
      </div>
      <div class="svg-container">
        <svg
          viewBox="0 0 1200 600"
          preserveAspectRatio="xMinYMin meet"
          class="svg-content"
        ></svg>
      </div>
    </section>

    <section class="inputs">
      <fieldset class="days">
        <h3>DAG</h3>
        <input
          type="radio"
          name="DAY"
          value="MAANDAG"
          id="MAANDAG"
          checked="true"
        />
        <label for="MAANDAG">MA</label>
        <input type="radio" name="DAY" value="DINSDAG" id="DINSDAG" />
        <label for="DINSDAG">DI</label>
        <input type="radio" name="DAY" value="WOENSDAG" id="WOENSDAG" />
        <label for="WOENSDAG">WO</label>
        <input type="radio" name="DAY" value="DONDERDAG" id="DONDERDAG" />
        <label for="DONDERDAG">DO</label>
        <input type="radio" name="DAY" value="VRIJDAG" id="VRIJDAG" />
        <label for="VRIJDAG">VR</label>
        <input type="radio" name="DAY" value="ZATERDAG" id="ZATERDAG" />
        <label for="ZATERDAG">ZA</label>
        <input type="radio" name="DAY" value="ZONDAG" id="ZONDAG" />
        <label for="ZONDAG">ZO</label>
      </fieldset>
      <fieldset class="time">
        <h3>TIJD <span></span></h3>
        <span>0:00</span>
        <input type="range" name="time" id="time" min="0" max="24" value="12" />
        <span>24:00</span>
      </fieldset>
      <fieldset class="citiesbox">
        <h3>GEMEENTES</h3>
        <fieldset class="cities"></fieldset>
      </fieldset>
    </section>
  </div>
</template>

<script>
import CleanData from "../components/CleanData.vue";
import * as d3 from "d3";

export default {
  name: "Chart",
  components: {
    myData: CleanData,
  },
  props: {
    data: Array,
  },
  mounted() {
    this.setSomeValuesFirst();
    this.createChart();
  },
  methods: {
    setSomeValuesFirst: function () {
      let day = "MAANDAG", time = 12,
          percentages = localStorage.getItem("percentages");
      if(localStorage.getItem("day")) day = localStorage.getItem("day");
      if(localStorage.getItem("time")) time = localStorage.getItem("time");
      document.querySelector("input[type=range]").value = time;
      document.querySelector("input[value=" + day + "]").checked = true;
      if (percentages == "true") {
        document.querySelector(".percentage input").checked = true;
        document.querySelector("#app").classList.add("perc");
      }
    },
    createChart: function () {
      let workArray = JSON.parse(localStorage.getItem("workArray"));
      let selectedCities;
      if (localStorage.getItem("selectedCities"))
        selectedCities = JSON.parse(localStorage.getItem("selectedCities"));
      else selectedCities = ["Amsterdam", "Delft", "Assen", "Den Haag"];
      // Set up some stuff. SVG = SVG, width=width etc.
      const svg = d3.select("svg");
      const width = 1200;
      const height = 600;
      //Get some margins otherwise the axis will fall outside the SVG.
      const margin = { top: 50, right: 50, bottom: 50, left: 100 };

      //Then calc an innerwidth and height.
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;

      // The yScale. The y axis and the data is run against this function so that everything is in proportion.
      let yScale = d3
        .scaleLinear()
        .domain([0, 8500]) // was set to d3.max(something) making the axis dynamic. I put this to 8500
        // (the max data output) so the relation between days is visually stronger
        .range([innerHeight, 0]); // upside down because svg draws from top left.

      // The xScale. The x axis and the data is run against this function so that everything is in proportion.
      // Citynames will be on the right spot, and the bars will have the correct width.

      const g = svg
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`)
        .attr("class", "barbox");

      createCityInputs(workArray);
      let combinedCityCapacityObjects = mergeCityCapacities(
        workArray,
        selectedCities
      );
      getCityTotals(workArray);
      makeGraphsWithD3(combinedCityCapacityObjects);

      function createCityInputs(data) {
        const citybox = document.querySelector(".cities");
        let allCityNames = [];
        for (let i in data) {
          if (!allCityNames.includes(data[i].stad)) {
            allCityNames.push(data[i].stad);
          }
        }
        for (let city in allCityNames) {
          let box = document.createElement("div");
          let input = document.createElement("input");
          input.setAttribute("type", "checkbox");
          input.setAttribute("value", allCityNames[city]);
          if (selectedCities.includes(allCityNames[city])) {
            input.setAttribute("checked", "checked");
          }
          let label = document.createElement("label");
          label.textContent = allCityNames[city];
          label.setAttribute("for", allCityNames[city]);
          box.append(input);
          box.append(label);
          citybox.append(box);
        }
      }

      function mergeCityCapacities(data, selectedCities) {
        let selectedCityObjects = data.filter((d) =>
          isCityMatch(d, selectedCities)
        );
        let openedCityObjects = selectedCityObjects.filter(isObjectOpened);

        return addCapacities(openedCityObjects, selectedCities);
      }

      function getCityTotals(data) {
        let allCityNames = [];
        for (let i in data) {
          if (!allCityNames.includes(data[i].stad)) {
            allCityNames.push(data[i].stad);
          }
        }

        let selectedCityObjects = data.filter((d) =>
          isCityMatch(d, allCityNames)
        );

        let mergedData = addCapacities(selectedCityObjects, allCityNames);

        localStorage.setItem("cityTotals", JSON.stringify(mergedData));
      }

      function isObjectOpened(cityObject) {
        const day = document.querySelector("input[type=radio]:checked").value,
          time = document.querySelector("input[type=range").value * 100;

        const objectDag = cityObject.openingstijden.find((d) => d.days == day);

        if (objectDag == null) return false;

        const openingsTijden = [objectDag.enterfrom, objectDag.enteruntil];
        if (time <= openingsTijden[1] && time >= openingsTijden[0]) return true;
        return false;
      }

      function addCapacities(cityObject, selectedCities) {
        let totals = [];
        for (let i in selectedCities) {
          totals[i] = { stad: selectedCities[i], capaciteit: 0, laadpalen: 0 };
          for (let city in cityObject) {
            if (selectedCities[i] == cityObject[city].stad) {
              totals[i].capaciteit += parseInt(cityObject[city].capaciteit);
            }
          }
        }
        return totals;
      }

      function isCityMatch(dataEntry, selectedCities) {
        for (let city in selectedCities) {
          if (selectedCities[city] == dataEntry.stad) {
            return true;
          }
        }
        return false;
      }
      function transformToPercentages(dataEntry) {
        const allCityTotals = JSON.parse(localStorage.getItem("cityTotals"));
        let percentage, i;
        for (i = 0; i < allCityTotals.length; i++) {
          if (allCityTotals[i].stad == dataEntry.stad) {
            percentage = parseInt(
              (dataEntry.capaciteit / allCityTotals[i].capaciteit) * 100
            );
          }
        }
        return { stad: dataEntry.stad, capaciteit: parseInt(percentage) };
      }

      function makeGraphsWithD3(workableData) {
        let percentages = localStorage.getItem("percentages");
        if (percentages == "true") {
          workableData = workableData.map(transformToPercentages);
          yScale = d3.scaleLinear().domain([0, 100]).range([innerHeight, 0]); //
        } else {
          yScale = d3.scaleLinear().domain([0, 8500]).range([innerHeight, 0]); //
        }
        const render = (data) => {
          const xScale = d3
            .scaleBand()
            .domain(data.map((d) => d.stad))
            .range([0, innerWidth])
            .padding(0.1);

          g.append("g").call(d3.axisLeft(yScale)).attr("class", "ax y");
          g.append("g")
            .call(d3.axisBottom(xScale))
            .attr("transform", `translate(0, ${innerHeight})`)
            .attr("class", "ax x");

          g.selectAll("rect")
            .data(data)
            .enter()
            .append("rect")
            .attr("x", (d) => xScale(d.stad))
            .attr("width", xScale.bandwidth())
            .attr("y", (d) => yScale(+d.capaciteit))
            .attr("height", (d) => innerHeight - yScale(+d.capaciteit))
            .attr("class", "bar");

          g.selectAll("text.bartext")
            .data(data)
            .enter()
            .append("text")
            .attr("class", "bartext")
            .attr("text-anchor", "middle")
            .attr("x", (d) => xScale(d.stad) + xScale.bandwidth() / 2)
            .attr("width", xScale.bandwidth())
            .attr("y", (d) => yScale(+d.capaciteit))
            .attr("transform", "translate(0, -10)")
            .text((d) => parseInt(+d.capaciteit));
        };
        //and put it to work!
        render(workableData);
      }

      function update(workableData) {
        let percentages = localStorage.getItem("percentages");
        if (percentages == "true") {
          workableData = workableData.map(transformToPercentages);
          yScale = d3.scaleLinear().domain([0, 100]).range([innerHeight, 0]); //
        } else {
          yScale = d3.scaleLinear().domain([0, 8500]).range([innerHeight, 0]); //
        }
        const render = (data) => {
          const xScale = d3
            .scaleBand()
            .domain(data.map((d) => d.stad)) // extracting the citynames from the dataset, making them the domain.
            .range([0, innerWidth]) // zero to 100
            .padding(0.1); // meaning distance between the bars

          let allBars = d3
            .select(".barbox")
            .selectAll("rect") // selecting the bars that are already made, putting them into a variable.
            .data(data); // put in the new data
          allBars
            .enter() // focus on what's new.
            .append("rect") // make a rect for this.

            // FROM
            .attr("x", width) // all the way to right because we want to make a nice transition, offscreen
            .attr("y", (d) => height - yScale(d.capaciteit)) // allllll the way to the bottom, offscreen
            .attr("width", xScale.bandwidth()) // get the width right though
            .attr("height", (d) => yScale(+d.capaciteit)) // height too
            .attr("class", "bar") // for me again
            .merge(allBars) // Putterthere!
            .transition() // and make it smooth
            .duration(500) // half a second

            //TO
            .attr("x", (d) => xScale(d.stad)) // new position
            .attr("y", (d) => yScale(+d.capaciteit)) //new position
            .attr("width", xScale.bandwidth()) // same width
            .attr("height", (d) => innerHeight - yScale(+d.capaciteit)); // same height

          allBars
            .exit() // focus on what's old.
            .remove(); // and get rid of that.

          //update texts
          let allTexts = g.selectAll("text.bartext").data(data);

          allTexts.text((d) => +d.capaciteit);
          
          allTexts
            .enter()
            .append("text")
            // FROM
            .attr("class", "bartext")
            .attr("text-anchor", "middle")
            .attr("transform", "translate(0, -8)")
            .attr("x", width)
            .attr("y", (d) => height - yScale(d.capaciteit))
            .text((d) => parseInt(+d.capaciteit))
            .merge(allTexts)
            .transition()
            .duration(500)

            // TO
            .attr("x", (d) => xScale(d.stad) + xScale.bandwidth() / 2)
            .attr("width", xScale.bandwidth())
            .attr("y", (d) => yScale(+d.capaciteit));

          allTexts.exit().remove();

          // update Axis
          g.selectAll(".ax.y")
            .transition()
            .duration(500)
            .call(d3.axisLeft(yScale));

          g.selectAll(".ax.x")
            .transition()
            .duration(500)
            .call(d3.axisBottom(xScale));
        };
        render(workableData);
      }

      // TIME SLIDER
      d3.select("#time").on("input", function () {
        let combinedCityCapacityObjects = mergeCityCapacities(
          workArray,
          selectedCities
        );
        localStorage.setItem("time", this.value);
        update(combinedCityCapacityObjects);
        updateTitle();
      });

      // CITIES CHECKBOXES
      d3.selectAll(".cities").on("click", function (e) {
        if (e.target.type == "checkbox") {
          if (e.target.checked) selectedCities.push(e.target.value);
          else
            selectedCities = selectedCities.filter((d) => d != e.target.value);
          localStorage.setItem(
            "selectedCities",
            JSON.stringify(selectedCities)
          );
          let combinedCityCapacityObjects = mergeCityCapacities(
            workArray,
            selectedCities
          );
          update(combinedCityCapacityObjects);
        }
      });

      // PERCENTAGE YES/NO
      d3.select(".percentage").on("change", function (e) {
        if (e.target.checked) {
          localStorage.setItem("percentages", true);
          document.querySelector("#app").classList.add("perc");
        } else {
          localStorage.setItem("percentages", false);
          document.querySelector("#app").classList.remove("perc");
        }
        let combinedCityCapacityObjects = mergeCityCapacities(
          workArray,
          selectedCities
        );
        update(combinedCityCapacityObjects);
      });

      // DAYS RADIOS
      d3.selectAll("input[type=radio]").on("change", function setDay() {
        let combinedCityCapacityObjects = mergeCityCapacities(
          workArray,
          selectedCities
        );
        localStorage.setItem("day", this.value);
        update(combinedCityCapacityObjects);
        updateTitle();
      });

      function updateTitle() {
        document.querySelector(".day").textContent = document.querySelector(
          "input[type=radio]:checked"
        ).value;
        let range = time_convert(
          document.querySelector("input[type=range]").value * 60
        );

        document.querySelector(".time").textContent = range;
        document.querySelector("h3 span").textContent = range;
      }

      // Source: https://www.w3resource.com/javascript-exercises/javascript-basic-exercise-51.php modified a bit myself
      function time_convert(num) {
        let hours = Math.floor(num / 60);
        let minutes = "00";
        if (hours < 10) hours = "0" + hours;
        return hours + ":" + minutes;
      }
      updateTitle();
    },
  },
};
</script>

<style scoped>
.svg-container {
  position: relative;
  width: 100%;
  padding-bottom: 50%;
  margin-top: 2rem;
}

.svg-content {
  display: inline-block;
  position: absolute;
  left: 0;
  top: 0;
}
div#chart {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  max-width: 70rem;
  width: calc(100% - 4rem);
  background: white;
}



section {
  display: flex;
  flex-direction: column;
  margin-top: 2rem;
}
h1,
h2 {
  margin-left: 2rem;
}
h2 span.day {
  text-transform: capitalize;
}

section.inputs > * {
  margin-top: 1rem;
}
section.inputs {
  height: 34rem;
  width: 12.6rem;
  position: relative;
}
section.inputs .days {
  height: 15%;
}
section.inputs .days label {
  user-select: none;
}
section.inputs .time {
  height: 15%;
  position: relative;
}
section.inputs .time > span {
  position: absolute;
  margin-top: 1rem;
  font-size: 0.7rem;
}
section.inputs .time > span:nth-of-type(1) {
  left: 0;
}
section.inputs .time > span:nth-of-type(2) {
  right: 0;
}
section.inputs .time > h3 span {
  font-size: 0.8rem;
  margin-left: 0.5rem;
  display: inline-block;
}

section.visualisation {
  width: 77%;
  align-items: flex-start;
  position: relative;
}

input[type="range"] {
  width: 12.6rem;
}
input[type="radio"] {
  display: none;
}

input[type="radio"] + label {
  color: #353535;
  border-radius: 0.2rem;
  padding: 0.2rem 0.1rem;
  font-size: 0.8rem;
  margin-right: 0.2rem;
  cursor: pointer;
  width: 1.4rem;
  display: inline-block;
  text-align: center;
}
input[type="radio"]:last-of-type + label {
  margin-right: 0rem;
}
input[type="radio"]:checked + label {
  background: steelblue;
  color: #fff;
  border-radius: 0.2rem;
  padding: 0.2rem 0.1rem;
  font-size: 0.8rem;
  transition: 500ms ease;
}
.perc input[type="radio"]:checked + label {
  background: maroon;
}

.citiesbox {
  height: 50%;
}
.cities {
  display: flex;
  flex-direction: column;
  height: 90%;
  overflow: scroll;
  background: white;
}
.cities label {
  font-size: 0.8rem;
  display: inline-block;
  margin-left: 1rem;
  user-select: none;
}
.cities > div {
  position: relative;
  display: flex;
  align-items: center;
  margin: 0.2rem 0;
}
.cities input {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0;
  cursor: pointer;
}
.cities label::after {
  border: 0.5px solid #353535;
  border-radius: 50%;
  width: 0.5rem;
  height: 0.5rem;
  display: block;
  content: "";
  opacity: 1;
  position: absolute;
  top: 0.35rem;
  left: 0.2rem;
  transition: 300ms ease;
}
.cities input:checked ~ label::after {
  background: steelblue;
  border: 0.5px solid transparent;
}
.perc .cities input:checked ~ label::after {
  background: maroon;
}
.cities input:hover ~ label::after {
  transform: scale(1.2);
  transform-origin: center;
}

.percentage {
  position: absolute;
  top: 8%;
  left: 50%;
  display: flex;
}

@media only screen and(max-width: 600px) {
  .percentage {
    top: 16%;
  }
}
.percentage input {
  position: absolute;
  top: 0%;
  left: 0%;
  width: 100%;
  height: 100%;
  opacity: 0;
  z-index: 3;
  cursor: pointer;
}
.percentage div {
  width: 2rem;
  height: 28px;
  background: #e9e9e9;
  border-radius: 28px;
  margin: 0 0.5rem;
  position: relative;
  display: flex;
  align-items: center;
  transition: 500ms ease;
}
.perc .percentage div {
  background: #aaa;
}

.percentage div::after {
  content: "";
  display: block;
  border-radius: 50%;
  background: steelblue;
  position: absolute;
  height: 0;
  width: 70%;
  padding-bottom: 70%;
  transition: 500ms ease;
  left: 0;
}
.percentage input:checked ~ div::after {
  transform: translateX(43%);
  background: maroon;
}


@media only screen and (max-width: 1200px) and (min-width: 800px) {
  div#chart {
    flex-direction: column;
  }
  div#chart .visualisation {
    width: 100%;
  }
  section.inputs {
    flex-direction: row;
    width: calc(100% - 2rem);
    justify-content: space-around;
    margin: 1rem;
  }
  input[type="range"] {
    width: 10rem;
  }
  .citiesbox {
    width: 25%;
    height: 50%;
  }
  .percentage {
    left: auto;
    right: 10%;
  }
}

@media only screen and (max-width: 800px) {
  div#chart {
    flex-direction: column;
      width: calc(100% - 2rem);

  }
  div#chart .visualisation {
    width: 100%;
  }
  section.inputs {
    margin: 0 auto;
  }
  .percentage {
    left: auto;
    right: 10%;
    transform: scale(0.7);
    transform-origin: right;
  }
  h1,
  h2 {
    margin-left: 0.5rem;
  }
  .cities {
    height: 70%;
  }
}
</style>
