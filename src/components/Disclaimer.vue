<template>
  <section class="mw">
    <div class="container">
      <h2>Disclaimer</h2>
      <ul>
        <li v-for="item in listItems" :key="item.message">
            {{ item.message }}
        </li>
      </ul>
    </div>
  </section>
</template>


<script>
export default {
  name: "Disclaimer",
  data: function () {
    return {
      listItems: [
        { message: "De data die is gebruikt voor het maken van de visualisatie komt van het RDW." },
        { message: "Niet alle parkeergelegenheden in Nederland hebben de voor deze visualisatie benodigde data beschikbaar. Hierdoor bestaat de mogelijkheid dat parkeergelegenheden buiten beschouwing zijn gelaten." },
      ],
    };
  },
};
</script>


<style scoped>
 h2 {margin-left: 0; font-family: "Bebas Neue"; font-size: 1.2rem; margin-top: 3rem;}
 ul { list-style-type: circle; }
 ul li {font-size: 0.5rem;}
</style>