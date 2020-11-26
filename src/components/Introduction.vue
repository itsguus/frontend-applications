<template>
    <section class="introduction">
        <div class="streetoverlay"></div>
        <hgroup>
            <h1>{{ h1 }}</h1>
            <h2>{{ h2 }}</h2>
        </hgroup>
        <img id="redcar" src="../assets/img/car.png" alt="An overhead view of a red car">
        <div class="streetlines">
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
        </div>
        <a href="#explanation"  class="arrow">
            <img src="../assets/img/arrow.svg" alt="arrow">
        </a>
    </section>
</template>


<script>
export default {
    name: "Introduction",
    data: function() {
        return {
            h1: "Parkeren in Nederland",
            h2: "Waar doe je dat nog?"
        }
    }
}
</script>


<style scoped>
    section.introduction {height: 200vh; width: 100vw; margin-top: 0; position: relative; background: #000;}
    section.introduction > * {z-index: 1;}
    div.streetoverlay {height: 100%; width: 100%; position: absolute; top: 0; left: 0;  background: url('../assets/img/floor.jpg') center center; background-size: 15rem; opacity: 0.7; z-index: 0;}
    hgroup {margin-top: 5%; position: sticky; top: 5%; bottom: 0; margin-bottom: 35%;}
    h1, h2 { color: white; text-align: center; margin: 0;}
    h1 { font-size: 5vw;}

    .introduction .streetlines { width: 100%; display: flex; justify-content: space-around; height: 32vw; position: absolute; bottom: 0; left: 0; opacity: 0.7}
    .introduction .streetlines span {display: block; width: 2%; height: 100%; background: #FFF;}
    .introduction img#redcar {width: 30%;  transform: rotate(90deg); position: sticky; top: 35%; margin-top: -20%;  bottom: 0; margin-bottom: 8%; margin-left: 4.5%;}


    .arrow { transform: rotate(90deg); width: 2rem; position: sticky; top: 90%; margin-bottom: 2rem; margin-right: 1rem; left: 95%; animation: bounce 3s ease infinite; margin-top: -10rem;}
    @keyframes bounce {
        0% {transform: rotate(90deg) translateX(0rem)}
        50% {transform: rotate(90deg) translateX(1rem)}
        100% {transform: rotate(90deg) translateX(0rem)}
    }


    @media only screen and (max-width: 600px) {
        h1 { font-size: 7vw;}
        section.introdcution { height: 120vh; }
    }
</style>