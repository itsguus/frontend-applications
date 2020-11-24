<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />
    <PackChart />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue';
import PackChart from  './components/Chart.vue';

import locationDataUrl from './assets/AreaIDwithlocationdata.json';
import parkingToegangUrl from  "./assets/parking-toegang.json"
import geoDataUrl from  "./assets/geodata.json"
import parkeerGebiedUrl from  "./assets/parkeergebied.json"


// import * as d3 from 'd3';


export default {
  name: 'App',
  components: {
    HelloWorld,
    PackChart
  },
  data() {
    return {
      loadData: {}
    };
  }, 
  mounted() {
    console.log("App loaded");
    this.fetchData();
  },
  methods: {
    async fetchData() {
    const self = this;

    let workArray = [];
    let listOfUsableAreaIds = [];




  listOfUsableAreaIds = setUpCleanDataSetWithIds(parkeerGebiedUrl);
  workArray = listOfUsableAreaIds;
  addDaysToWorkArray(parkingToegangUrl);
  addCapacityToWorkArray(parkeerGebiedUrl);
  addCityNameToWorkArray(geoDataUrl)




// let myObject = [];

function addCityNameToWorkArray() {
    // let areaIdsAndGeoData = geoDataSet.map(entry => [entry.areaid, { geodata: entry.areageometryastext }]),
    //     areaIdsAndGeoDataThatIWant = areaIdsAndGeoData.filter(checkIfAreaIdInWorkArray);

    // areaIdsAndGeoDataThatIWant = areaIdsAndGeoDataThatIWant.map(shapeToLatLong);

    // // let cityNames = areaIdsAndGeoDataThatIWant.map(translateLatLongToCityName);

    // // collectData(areaIdsAndGeoDataThatIWant, 0);


    const dataSet = locationDataUrl;
    let filteredSet = dataSet.filter(isObjectFilled);
    let areaIDWithCityName = filteredSet.map(cityDataToCityName);
    workArray = areaIDWithCityName.map(entry => transformSingleEntry(entry, workArray));
    workArray = workArray.filter(entry => !isNull(entry));
    workArray = workArray.map(reformatDataEntry);
    self.loadData = workArray;
}


function reformatDataEntry(dataEntry) {
    const weekdays = dataEntry[1],
        capacity = dataEntry[2].capacity,
        laadPalen = dataEntry[2].laadPalen;
    let cityName = dataEntry[3].cityname

    if (cityName == null) cityName = dataEntry[4].cityname

    return { stad: cityName, capaciteit: capacity, openingstijden: weekdays, laadpalen: laadPalen };
}


function isObjectFilled(object) {
    if (object[1].cityname == null) return false;
    return true;
}


function cityDataToCityName(cityObject) {
    let cityData = cityObject[1].cityname,
        areaID = cityObject[0],
        cityName = "";

    if ('town' in cityData) cityName = cityData.town;
    else if ('city' in cityData) cityName = cityData.city;
    else cityName = cityData.village;

    return [areaID, { cityname: cityName }];
}

function setUpCleanDataSetWithIds(dataSet) {
    return dataSet.map(entry => entry["areaid"]);
}


function addCapacityToWorkArray(dataSet) {
    let newData = dataSet.map(entry => [entry.areaid, { capacity: entry.capacity, laadPalen: entry.chargingpointcapacity }]);
    // output: 
    // capacity: x
    // laadPalen: y

    workArray = newData.map(entry => transformSingleEntry(entry, workArray));
    workArray = workArray.filter(entry => !isNull(entry));
}


function transformSingleEntry(entryToAdd, dataSetToBeAddedTo) {
    const entryToAddId = entryToAdd[0],
        entryToAddObjs = entryToAdd[1];

    if (dataSetToBeAddedTo.find(entry => entry[0] == entryToAddId)) {
        let matchedArray = dataSetToBeAddedTo.find(entry => entry[0] == entryToAddId);
        matchedArray.push(entryToAddObjs)
        return matchedArray;
    }
    return null;
}

function isNull(dataEntry) {
    if (dataEntry == null) return true;
    return false;
}


function addDaysToWorkArray(dataSet) {
    let allAreaIdsPerDay = (Object.entries(workArray = groupByKey(dataSet, "areaid")));
    workArray = allAreaIdsPerDay.map(entry => deleteExcessData(entry));
}


function deleteExcessData(parkingArea) {
    let weekdays = parkingArea[1]
    for (let day in weekdays) {
        delete weekdays[day].areamanagerid;
        delete weekdays[day].areaid;
        delete weekdays[day].startofperiod;
    }
    return parkingArea;
}



// https://stackoverflow.com/questions/40774697/how-to-group-an-array-of-objects-by-key
function groupByKey(array, key) {
    return array
        .reduce((hash, obj) => {
            if (obj[key] === undefined) return hash;
            return Object.assign(hash, { [obj[key]]: (hash[obj[key]] || []).concat(obj) })
        }, {})
}




    }
  }
};
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>